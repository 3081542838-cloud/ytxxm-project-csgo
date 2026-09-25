# 兼容性状态

日期：2026-09-26。以下“通过”只表示所列具体能力，不等于 App 或导出流程已可用。

| 环境/样本 | 启动与 hook | Demo 控制 | 玩家锁定/预览 | MP4 120/240 |
| --- | --- | --- | --- | --- |
| 本机 CS2 build 25515854、PatchVersion 1.41.8.4；HLAE 2.192.4；完美平台单 DEM ZIP | 通过 | 加载、play、pause、基础 seek 通过 | 未验证 | 未验证 |
| 其他 CS2/HLAE 组合 | 未验证 | 未验证 | 未验证 | 未验证 |
| 5E Demo | 未验证 | 未验证 | 未验证 | 未验证 |

CS2 自动更新可能改变接口。HLAE 2.192.4 [发布说明](https://github.com/advancedfx/advancedfx/releases/tag/v2.192.4)只写明适配 `1.41.8.3`；本机实际为 `1.41.8.4`，故应按能力探测，而非版本号一刀切。HLAE 启动和 hook 通过不代表指定玩家视角、画面捕获或录制通过。

Windows Graphics Capture 的按窗口接口由 Microsoft 文档列为 Windows 10 1903 起可用，但本项目尚未在 Windows 10/11 做实机捕获和便携包验收：[CreateForWindow](https://learn.microsoft.com/en-us/windows/win32/api/windows.graphics.capture.interop/nf-windows-graphics-capture-interop-igraphicscaptureiteminterop-createforwindow)。
