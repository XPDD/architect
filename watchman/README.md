# Watchman 技术调研笔记

> 资料：https://facebook.github.io/watchman/ 及相关子页面
> 时间：2026-09-15
> 说明：本目录与架构师学习无关，是 sunwu-claw「KB 工作区文件变化监听」场景的技术调研与集成方案沉淀。

## 一、Watchman 是什么

Watchman 是 Meta 开源的**文件监听服务**（file watching service）。

- 常驻守护进程（daemon），不是库
- 递归监听一个或多个目录树（称 root）
- 记录文件变化，可查询「某时间点之后哪些文件变了」
- 可在文件变化时触发动作（trigger），如重新构建、报警、同步
- 跨平台：Linux(inotify) / macOS(FSEvents) / Windows(ReadDirectoryChangesW)

安装：`apt install watchman` / `brew install watchman`

## 二、第一性原理

Watchman 对「文件监控」问题做了 5 条本质抽象：

1. **把「变化」建模成一条时间线（clock）**
   不是一堆孤立事件，而是单调递增的时钟。任何时刻可问「自 clock N 以来哪些文件变了」。变化历史 → 可查询、可对比、可增量。

2. **监控是独立服务，不是库**
   多客户端共享一份 watch 成本；watch 状态跨重启持久化；大仓库只扫一次。

3. **核心能力是「回答查询」，不是「推事件」**
   持久化整棵树的元数据（name/size/mtime/exists），`subscribe` 只是「定期 query 的变化版」。查询模型是内核，事件推送只是壳。

4. **trigger = 订阅 + 表达式 + 动作**
   文件满足表达式 → 等文件系统 settle（避开编辑器临时文件+rename 中间态）→ 批量喂给命令，单实例串行防 fork-bomb。

5. **保守一致性**
   不确定就当变化，宁可多报不漏报；先 settle 再报告。牺牲「精确」换取「不错过」。

**一句话**：文件变化检测的本质是「你怎么知道它变了」——要么等内核通知（事件），要么自己反复去看（轮询）。**Watchman 优化的是「等内核通知」这条路径。**

## 三、核心概念与命令

| 概念 | 说明 | 关键命令 |
|---|---|---|
| root | 被监听的目录树 | `watchman watch <dir>` / `watch-project` |
| clock | 单调时钟，变化时间线 | `watchman clock <root>` |
| query | 查询变化集 / 当前状态 | `watchman query <root> ...` |
| subscribe | 订阅实时变化（连接期间有效） | `subscribe` / `unsubscribe` |
| trigger | 变化时执行命令（持久化，重启恢复） | `-- trigger <root> <name> [patterns] -- <cmd>` |

### trigger 用法（扩展语法 + stdin）

```sh
watchman -j <<-EOT
["trigger", "/opt/data", {
  "name": "kb-reconcile",
  "expression": ["allof", ["type", "f"], ["not", ["name", ".env"]]],
  "command": ["/opt/sunwu/kb-reconcile-hook.py"],
  "stdin": ["name", "exists"],
  "max_files_stdin": 500
}]
EOT
```

- `stdin: ["name","exists"]` → 变化文件以 JSON 数组喂给 command 的 stdin，如 `[{"name":"output/a.md","exists":true}]`
- 删除文件也算变化（`exists:false`）
- trigger 只 spawn 本地进程，**不原生支持 HTTP**；要发 HTTP 得让 command 本身是 curl / hook 脚本

### 客户端形态

- **CLI**：最常用，`watchman` 客户端/服务端一体
- **C++ Client**（`WatchmanClient.h`）：重度依赖 Facebook Folly，仅构建时带 Folly 才生成
- **NodeJS**（`fb-watchman`）：npm 包，走 Unix socket + JSON/BSER
- **Socket 接口**：任何语言可连 Unix socket 发 JSON 数组命令，`subscribe` 后持续读变化

## 四、关键局限：NFS / 网络文件系统

**Watchman 感知变化的唯一渠道是内核文件系统事件（inotify/kqueue/FSEvents）。**

- inotify 事件在 **VFS 层**产生，只覆盖**本内核**发生的写入
- NFS（如阿里云 NAS）**跨节点写入发生在远端内核**，本节点内核不产生任何事件 → Watchman 完全感知不到
- 官方自 2016 年 issue #201 起想加「针对 NFS 的 polling watcher」，至今未实现

**推论**：当事件输入端失效（NFS 跨节点），Watchman 的整条「事件 → 时间线 → 订阅/触发」链路从地基就是空的。它不是「监控文件变化」的通用答案，而是「在事件可达的本地文件系统上高效监控」的答案。

## 五、sunwu-claw 场景分析

### 现状

- `kb-doc-reconcile` 每 5 分钟（现改 1 分钟）scan BFF `/api/internal/kb-documents/scan` → 读工作区文件 sha256 → 与 `content_hash` 对比 → 不一致则 resync
- 工作区 `/opt/data` 挂载**阿里云 NAS PVC**（ReadWriteMany，`alicloud-nas-subpath`）
- 写入来源分散：agent 工具（Hermes 同 pod）、资源中心上传（网关同 pod）、**沙箱 shell（独立 pod，不同节点）**、云浏览器下载（browser sidecar 不同节点）

### 事件可达性矩阵

| 写入来源 | 事件可达？ | 兜底 |
|---|---|---|
| agent 工具写（Hermes 同 pod） | ✓ | watchman / hook |
| 资源中心上传（网关同 pod） | ✓ | watchman |
| 沙箱 shell 写（不同节点） | ✗ | 轮询 |
| 云浏览器下载（不同节点） | ✗ | 轮询 |

### 结论

- **Watchman/fsnotify 只覆盖同节点写入**，沙箱等跨节点写入靠轮询兜底
- **Hermes hook（`edit_file` 等工具）在 Hermes 进程内触发，与节点无关**，是「agent 工具写入」最可靠的主动触发方式
- 轮询（stat mtime/size 快扫，比 sha256 便宜）是唯一稳定兜底，与存储类型无关

## 六、方案对比

| 方案 | 实时性 | NFS 可靠性 | 依赖/复杂度 | 结论 |
|---|---|---|---|---|
| 纯 fsnotify（inotify） | 实时 | ✗ 跨节点丢事件 | 低 | 需轮询兜底 |
| watchman（trigger/socket） | 近实时 | ✗ 跨节点丢事件 | 中（多守护进程） | 需轮询兜底 |
| **轻量轮询 mtime+size** | 15~30s | ✓ 与存储无关 | 低 | **推荐兜底** |
| **Hermes hook（工具写入）** | 即时 | ✓ 与节点无关 | 低 | **推荐主动触发** |
| 自建 HTTP 监听 API（fsnotify+轮询） | 秒级+兜底 | ✓ | 低（纯 Go 增量） | 若需对外 API，推荐 |

### 集成建议（若坚持用 watchman）

1. `deploy/Dockerfile` 加 `apt install watchman`
2. s6 注册启动脚本：`watchman watch /opt/data` + trigger（持久化，重启自动恢复）
3. trigger 的 command = curl 或 hook.py，POST 网关 `http://<instance>:18643/...`
4. 网关加定向核对 handler（复用 fetchKbDocScan + sha256 + resync）
5. **保留轮询兜底**（跨节点写入）
6. 可观测：trigger 输出重定向日志；hook 结构化日志 + 事件覆盖率对比轮询

### 二次开发不建议

- Watchman 是 C++ + Folly，改 watcher 抽象层 = 重新构建 + 维护 fork
- 若真需要 NFS polling watcher，在网关 Go 里自写 ~50 行 stat 轮询更划算

## 七、参考链接

- 主页：https://facebook.github.io/watchman/
- CLI：https://facebook.github.io/watchman/docs/cli-options
- trigger：https://facebook.github.io/watchman/docs/cmd/trigger
- C++ Client：https://facebook.github.io/watchman/docs/cppclient
- NodeJS Client：https://facebook.github.io/watchman/docs/nodejs