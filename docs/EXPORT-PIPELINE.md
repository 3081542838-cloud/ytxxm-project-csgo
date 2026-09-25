# MP4 导出流程与验收门槛

状态：**设计，未实现、未验证**。必须先完成指定玩家第一人称与 App 内真实预览。

## 固定输出

默认 1920×1080、120 FPS、High（CRF 16），H.264 `libx264`、`preset=medium`、`yuv420p`、MP4；音频存在时 AAC 192 kbps、48 kHz。其他 FPS：30、60、144、240；分辨率：720p、1080p、1440p、4K；画质：High/CRF 16、Balanced/CRF 20、Small/CRF 24。界面不让普通用户输入 FFmpeg 参数。

预览 `previewFps`、HLAE 录制 `renderFps`、成片 `outputFps` 独立保存。正常高帧率模式下 `renderFps=outputFps`；预览可以是 60 FPS。240 FPS 成功必须来自 CS2/HLAE 实际生成足够独立画面，不用 `-r 240` 重复 60 FPS 帧，也不用 AI 插帧。

## 作业顺序

1. 校验依赖和能力：HLAE hook、FFmpeg/FFprobe、libx264、AAC、MP4。
2. 校验源 Demo 指纹、选定玩家稳定 ID、`inTick < outTick`、预设/语音和磁盘空间。
3. 在 `data/temp/render/{jobId}` 创建唯一临时目录；不得覆盖已有最终 MP4。
4. 加载 Demo、选择玩家、设置第一人称及正常 HUD、应用已验证的视觉/语音设置。
5. 跳到预滚位置并观察完成，再推进至入点。HLAE 开始录制，到出点停止。进度来自实测 tick/帧数；无法计算时显示阶段状态。
6. FFmpeg 在临时目录编码；FFprobe 验证容器、视频编码、尺寸、帧率、时长、帧数和音频状态。无音频的 Demo 不因此失败。
7. 仅在验证通过后安全移动至用户目标目录；若已有同名文件，要求改名或明确确认覆盖。失败或中断不得标记 Completed。

## 分阶段证据

先导出 10 秒 1080p120，约 1200 帧；核实真实第一人称、HUD、时间范围、音频，以及画面中无 Demo Bar、控制台、HLAE UI、App UI。再做 10 秒 1080p240，约 2400 帧；除 FFprobe 的 `avg_frame_rate`/`r_frame_rate`/`nb_frames` 外，须检测是否只是简单重复帧。边界帧允许少量误差，不能仅凭 MP4 元数据判定成功。

随后逐项验证 720p60、1080p60/120/144/240、1440p120/240、4K60/120。4K240 只作为实验，不承诺所有硬件稳定。

## 依据

- [HLAE Source2 `mirv_streams` 实际录制接口](https://github.com/advancedfx/advancedfx/wiki/Source2:mirv_streams)
- [FFmpeg 官方文档](https://ffmpeg.org/documentation.html)
