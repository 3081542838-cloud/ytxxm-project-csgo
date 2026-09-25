# 本地与开源安全边界

## Demo 与输出

- 样本 DEM/ZIP、文件名、原始路径、视频、日志及应用 `data/` 不进入公开仓库。`.gitignore` 拦截常见扩展名，提交前仍需人工检查 `git status` / `git diff --cached`。
- 输入 ZIP 必须恰有一个 DEM，并阻止绝对路径、盘符路径、`..`、链接与 canonical path 逃逸。解压只写到新建的 job 专属临时目录；源文件永不修改。
- 输出先写临时目录，验证后移入目标路径。已有目标默认拒绝覆盖。配置和项目 JSON 采用临时文件、fsync、rename 原子保存，并带 `schemaVersion`。

## HLAE/CS2 隔离

- 只使用官方渠道获取 HLAE。App 不静默下载任何第三方依赖，不把 Steam、CS2、HLAE 或 FFmpeg 二进制提交仓库。
- 制作会话强制 `-insecure`、独立 `USRLOCALCSGO` 配置及 `-afxDisableSteamStorage`。不编辑 GameInfo、正常 CS2 配置或游戏安装文件。启动前若普通 CS2 已运行，提示用户关闭；App 不结束该进程。
- 只做本地 Demo 回放；不提供服务器地址、connect、匹配或社区服入口。首次使用告知 HLAE 仅供离线 Demo 制作，不要用此实例连接 VAC 服务器。参见 [HLAE FAQ](https://github.com/advancedfx/advancedfx/wiki/FAQ)。
- 本次 Spike 使用 `127.0.0.1` 命令端口；后续正式实现须随机空闲端口、验证会话归属并限制命令类型，避免任意 console command IPC。

## Electron 与任务

Electron 目标配置：`contextIsolation=true`、`nodeIntegration=false`。renderer 不直接读写文件、创建进程、调用 Win32 或传任意控制台命令；preload 只暴露 `seekTick`、`selectPlayer` 等强类型 API，main 验证全部参数并生成固定命令。

RenderJob 每阶段有超时、成功和失败条件及独立日志。异常退出时恢复为 Interrupted，不能推断 Completed。日志默认本地保存，公开 issue/日志分享前需脱敏玩家身份、Demo 文件名和路径。

## 许可证

项目自有代码拟采用 MIT；该许可仅覆盖本项目代码。Steam、Counter-Strike 2、HLAE、FFmpeg 各遵循其自身许可和官方渠道。正式发布前应加入 MIT LICENSE 文件及第三方声明。
