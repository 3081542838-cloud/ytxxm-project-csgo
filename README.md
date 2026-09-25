# CS2 Demo 本地 MP4 导出工具

目标是让用户在比赛结束后导入 CS2 Demo，在 Windows 桌面 App 中预览自己的第一人称视角、选择片段，再从游戏回放离线生成 MP4。比赛时无需提前录屏；Demo 与项目数据留在本机。

**当前状态：技术 Spike，尚无可下载 App 或已验证的 MP4 导出。** 本机已验证 HLAE 自动启动、hook 注入、完美平台样本加载以及基本 play/pause/seek。单一样本玩家视角和微软官方 WGC 示例已初步通过；本项目 App 内预览、120/240 FPS 导出均未验证，不能视为可用。

开发顺序和实测记录见 [架构](docs/ARCHITECTURE.md)、[CS2/HLAE Spike](docs/SPIKE-CS2-HLAE.md)、[导出流程](docs/EXPORT-PIPELINE.md)、[兼容性](docs/COMPATIBILITY.md)、[安全边界](docs/SECURITY.md)。

未来正式版计划提供 Windows 便携包；用户需自行安装 Steam、CS2、HLAE、FFmpeg。本仓库不包含这些第三方程序，也不包含任何 Demo 样本。

项目自有代码采用 [MIT 许可](LICENSE)；该许可不覆盖 Steam、CS2、HLAE、FFmpeg。
