# 架构与实施顺序

日期：2026-09-26。当前仓库仍处于技术 Spike 阶段，尚无 Electron 应用。不要将本文的目标架构视为已实现功能。

## 产品边界

Windows 10 1903+ / Windows 11 便携桌面应用；所有项目数据留在本机。用户自行安装 Steam、CS2、HLAE 和 FFmpeg。比赛结束后导入一个 `.dem` 或只含一个 `.dem` 的 ZIP，选择本人玩家与片段，App 显示真实 CS2 窗口预览，最终由 HLAE 离线录制、FFmpeg 编码为 MP4。不得用桌面录屏或伪视频代替 CS2/HLAE。

## 开发门槛

1. DemoIndexer 只读解析样本，返回地图、玩家稳定 ID、回合、demo tick 范围及语音能力。外部解析器 Spike 已取得地图、玩家 ID、回合事件；项目实现及精确 tickRate/总 tick/语音能力未验证。
2. Cs2SessionController 证明 HLAE hook、Demo 加载、指定玩家第一人称、播放、暂停和定位均可由 App 控制。当前已证实 hook、Demo 加载、播放/暂停/基础定位，以及单一样本目标的第一人称；用户所选本人视角在 seek 后的稳定性未证实，见 [Spike](SPIKE-CS2-HLAE.md)。
3. PreviewAdapter 通过 Windows Graphics Capture + CS2 HWND 取得真实非黑屏游戏帧，并显示在 App 窗口内。微软官方独立示例已捕获 CS2 HWND；本项目 App 集成未验证。
4. RenderJob 先真实生成并验收 10 秒 1080p120，再真实生成 1080p240。未开始。
5. 门槛全部通过后，才完善设置、天空/X-Ray/语音、完整三栏 UI、持久化与便携打包。

## 目标模块

| 模块 | 边界 |
| --- | --- |
| DependencyService | 查找并验证 Steam、CS2、HLAE、AfxHookSource2、FFmpeg、FFprobe 的路径、版本和能力；找不到时让用户选择，不静默下载。 |
| DemoIndexer | 验证 ZIP 安全、读取 DEM、建立 demo tick 索引、玩家稳定 ID、回合与能力；不控制 CS2。 |
| Cs2SessionController | 专用 `-insecure` 会话，状态机，超时、成功/失败条件和日志；命令从受限参数构建，不接收任意控制台字符串。 |
| PreviewAdapter | 对指定 HWND 使用 Windows Graphics Capture；捕获前验证窗口不是最小化且尺寸有效，最小化时提示恢复；预览 FPS 与导出 FPS 分开。 |
| RenderJob | 校验选段及空间，HLAE 录制，FFmpeg 编码，FFprobe 验证，成功后安全移入目标目录。 |
| ProjectStore | `data/` 内 JSON 原子写入，schemaVersion、源文件指纹、任务中断恢复状态。 |
| Electron UI | renderer 仅经 typed IPC 调用 main 中的明确操作。`contextIsolation=true`，`nodeIntegration=false`。 |

## 状态机

会话：Idle → Starting → WaitingForGame → HookReady → LoadingDemo → DemoReady → Seeking / Playing / Paused → Rendering → Stopping；任一步可进入 Failed。必须由实际反馈转移，不以固定 sleep 宣称完成。

导出：Created → Preparing → Launching → LoadingDemo → Seeking → Rendering → Encoding → Finalizing → Verifying → Completed；失败、取消及崩溃恢复分别为 Failed、Cancelled、Interrupted。

## 第一版本人视角

导入后显示玩家昵称供用户点选自己，项目保存该玩家的 XUID/SteamID。只为这名玩家寻找当前 CS2 控制器索引并锁定第一人称；每次加载或跳转后检查实际观察目标是否一致。若昵称映射不唯一或目标不一致，停止预览/导出并提示重新选择，不猜测、不导出错误视角。无需映射或支持所有其他玩家。

## 精确时间

持久化 `inTick`、`outTick`、`currentTick`，明确它们是 **demo tick**。这次实测 `demo_gototick 16000` 的响应同时出现 game tick 19332/19333；两种 tick 不能互换。UI 的 HH:MM:SS.mmm 从可验证的 demo tick 映射生成，不作为精确数据源。

## 当前设计选择

默认 1920×1080、120 FPS、High；FPS 候选 30/60/120/144/240。预览目标可为 60 FPS，但必须与离线录制和输出 FPS 解耦。完整默认与验收条件见 [导出流程](EXPORT-PIPELINE.md)。

## 依据

- [HLAE Custom Loader 命令行](https://github.com/advancedfx/advancedfx/wiki/HLAE-Interfaces)
- [HLAE CS2 启动与隔离配置](https://github.com/advancedfx/advancedfx/wiki/AfxHookSource2)
- [Windows Graphics Capture 按窗口创建](https://learn.microsoft.com/en-us/windows/win32/api/windows.graphics.capture.interop/nf-windows-graphics-capture-interop-igraphicscaptureiteminterop-createforwindow)
