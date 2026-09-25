# CS2 / HLAE 实机技术 Spike

日期：2026-09-26。以下只记录实测，不把后续计划写为已通过。

## 本机依赖

| 项目 | 结果 |
| --- | --- |
| Steam | `D:\steam\Steam.exe`；当时正在运行 |
| CS2 | `D:\steam\steamapps\common\Counter-Strike Global Offensive\game\bin\win64\cs2.exe` |
| CS2 build | Steam appmanifest buildid `25515854`；`steam.inf` PatchVersion `1.41.8.4`、ClientVersion `2000917`、VersionDate `Sep 24 2026` |
| HLAE | 便携版 `2.192.4.0`，本机 Spike 路径 `D:\CS2Spike_20260926\hlae\HLAE.exe` |
| AfxHookSource2 | `D:\CS2Spike_20260926\hlae\x64\AfxHookSource2.dll`；HLAE release 内版本 `0.41.4` |
| FFmpeg / FFprobe | 本机未发现；HLAE 便携 ZIP 的 `ffmpeg` 目录仅有说明文件，未自带可执行文件。版本、路径及编码能力均未验证。 |
| HLAE 下载校验 | 官方 release ZIP 长度 8,997,657 bytes；SHA-256 `0718ADFBB5E2786A85D454262A264892B94AFFBA25ED07B8D9A97A5F57DCAAE8`。这是本次下载的校验值，并非声称官方公布的签名。 |

HLAE 2.192.4 的[发布说明](https://github.com/advancedfx/advancedfx/releases/tag/v2.192.4)写明适配 CS2 `1.41.8.3`。本机为 `1.41.8.4`。实际 hook、Demo 控制在此组合上部分通过；不能因此推断其他功能也兼容。

## 已执行操作

1. 检查启动前没有 `cs2.exe`。从 HLAE 官方 ZIP 解压到新的英文临时目录；不覆盖 CS2 安装文件。
2. 按 [HLAE 官方 Custom Loader 接口](https://github.com/advancedfx/advancedfx/wiki/HLAE-Interfaces)运行 `-customLoader -noGui -noConfig -autoStart -hookDllPath <AfxHookSource2.dll> -programPath <cs2.exe> -cmdLine <options> -addEnv USRLOCALCSGO=<isolated-config>`。`options` 为 `-steam -insecure +sv_lan 1 -console -sw -w 1280 -h 720 -afxDisableSteamStorage -afxFixNetCon -netconport 21212 +playdemo "<temporary-demo-path>"`。具体 Demo 文件名/原路径不进入公开仓库。
3. 源 ZIP 只读检查：恰好一个 DEM；临时解压副本 89,491,937 bytes，前 7 字节 `PBDEMS2`。原 ZIP 未改动。
4. HLAE 退出码 0 后检查独立的 `cs2.exe`：窗口标题 `Counter-Strike 2`，窗口句柄非零，进程模块列表含本次 HLAE 的 `AfxHookSource2.dll`，启动命令含 `-insecure`。HLAE 退出码本身不作为 hook 成功证据。
5. TCP `127.0.0.1:21212` 可连，发送 ASCII `echo SPIKE_NETCON_2` 后收到同一标记，证明本地命令通道有实际回显。
6. `demo_info` 返回 `DemoFileHeader: demo_file_stamp: "PBDEMS2\000"`、`client_name: "SourceTV Demo"`、`playback_time: 1899.84375`。`demo_pause` 返回 `paused on tick 8740`；`demo_resume` 后再次暂停返回 `paused on tick 8964`。
7. 发送 `demo_gototick 16000` 后，回显 `Demo Skipping finished at tick 16000`；随后暂停显示 game tick `19333`。该差异需要在索引器和时间轴中特别处理。
8. `find spec_mode`、`find spec_player`、`find spec_next` 能列出命令；`find spec_lock_to_accountid` 与 `find mirv_spectate` 未列出可用锁定命令。发送 `spec_mode 1` 后，对专用 CS2 窗口做本地诊断截图，确实看到游戏地图、玩家武器和正常 HUD；这是窗口画面核查，不是 Windows Graphics Capture 或 App 内预览。截图含玩家信息，未放入仓库。首次截图尝试时游戏窗口处于最小化状态（GetWindowRect 仅约 158×26），恢复窗口后才取得游戏画面；正式预览需处理最小化/恢复状态。
9. HLAE 随包的 `mirv_script_spec_lock.js` 可以加载，`list` 返回 11 个当前玩家控制器索引。发送 `mirv_script_spec_lock 4` 无命令错误，诊断画面改变；下述只读交叉检查确认了该索引对应的观察目标，但尚未证明任意玩家或 seek 后稳定。此脚本按实体索引每帧重复执行 `spec_player`。`mirv_deathmsg help players` 给出 XUID，脚本 `list` 给出索引；在此样本中，按唯一姓名可将 10 个 XUID 中的 9 个匹配到索引，1 个仍未匹配，不能把姓名当通用稳定映射。对索引 4，已匹配唯一 XUID，HLAE 官方 `observer-test.js` 报告观察目标姓名与索引 4 相同、Observer mode 为 2 且武器存在。之后一次 seek 后仍观察到该目标与 mode 2，但那次 seek 未收到明确完成回执，因此不算 seek 后稳定性验收。

## 结果矩阵

| 能力 | 状态 | 证据/限制 |
| --- | --- | --- |
| HLAE 自动启动 CS2 | 通过 | 专用 CS2 进程、窗口、`-insecure` 参数均出现 |
| AfxHookSource2 hook | 通过 | CS2 进程模块列表含正确 DLL |
| 完美平台 Demo 加载 | 通过 | `demo_info` 返回正确魔数与 1899.84375 秒时长 |
| Play | 通过 | Pause tick 8740 → Resume → Pause tick 8964 |
| Pause | 通过 | 游戏控制台明确回报暂停 tick |
| Seek | 基础命令通过 | `demo_gototick 16000` 完成；尚未验证任意 tick 精度和画面同步 |
| 指定玩家锁定 | 单一样本目标通过，整体未验收 | 索引 4 对应唯一 XUID；官方观察脚本报告目标一致、mode 2、有武器。另 1 个 XUID 尚未映射，多目标与 seek 后稳定性未证实。 |
| 第一人称画面 | 基础视觉通过 | 本地窗口诊断图显示真实地图、武器与 HUD；尚未核实指定玩家，也不是 App 内预览。 |
| Windows Graphics Capture / App 内预览 | 未验证 | 尚无 native helper 或 App。 |
| 非黑屏 | 未验证 | 尚无真实帧捕获。 |
| 1080p120 / 1080p240 导出 | 未验证 | 缺 FFmpeg，且预览与玩家锁定门槛未过。 |
| FFmpeg 版本与编码能力 | 未验证 | 可执行文件尚未定位。 |

## 风险及下一步

HLAE 项目[玩家视角锁定问题 #1172](https://github.com/advancedfx/advancedfx/issues/1172)记录：CS2 `1.41.6.2` 更新后，`spec_player` 等命令虽然存在，却可能无法按参数稳定选择目标。该报告针对旧版本，不能直接断定本机 `1.41.8.4` 仍有同一故障；必须用本地 Demo 的稳定玩家 ID、实际第一人称画面和多次 seek 复测。问题关闭状态也不能代替功能验证。

下一个工程门槛：只读解析样本玩家稳定 ID；建立每个 ID 到当前实体索引的可验证映射，使用 HLAE 官方 `mirv_script_spec_lock.js` 反复锁定指定玩家；用 Windows Graphics Capture 获取 CS2 HWND 的帧，确认人称、非黑屏、HUD 与 seek 同步。若映射或锁定不可靠，需要重新评估产品路线，不能靠随机 `spec_next` 或伪预览宣称完成。

此时仓库无 `package.json`、测试或构建脚本；typecheck、tests、build 均不适用，未声称通过。
