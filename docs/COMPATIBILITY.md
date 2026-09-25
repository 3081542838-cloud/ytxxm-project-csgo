# 兼容性状态

日期：2026-09-26。以下“通过”只表示所列具体能力，不等于 App 或导出流程已可用。

| 环境/样本 | 启动与 hook | Demo 控制 | 玩家锁定/WGC/本项目预览 | MP4 120/240 |
| --- | --- | --- | --- | --- |
| 本机 CS2 build 25515854、PatchVersion 1.41.8.4；HLAE 2.192.4；完美平台单 DEM ZIP | 通过 | 加载、play、pause、基础 seek 通过 | 单一目标锁定与微软 WGC 示例通过；本项目预览未验证 | 未验证 |
| 其他 CS2/HLAE 组合 | 未验证 | 未验证 | 未验证 | 未验证 |
| 5E Demo | 未验证 | 未验证 | 未验证 | 未验证 |

CS2 自动更新可能改变接口。HLAE 2.192.4 [发布说明](https://github.com/advancedfx/advancedfx/releases/tag/v2.192.4)只写明适配 `1.41.8.3`；本机实际为 `1.41.8.4`，故应按能力探测，而非版本号一刀切。HLAE 启动和 hook 通过不代表指定玩家视角、画面捕获或录制通过。

Windows Graphics Capture 的按窗口接口由 Microsoft 文档列为 Windows 10 1903 起可用，本机 Windows 11 已用微软官方独立示例实测 CS2 HWND 捕获；本项目 App 集成、Windows 10 与便携包尚未验收：[CreateForWindow](https://learn.microsoft.com/en-us/windows/win32/api/windows.graphics.capture.interop/nf-windows-graphics-capture-interop-igraphicscaptureiteminterop-createforwindow)。
