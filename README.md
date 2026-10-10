# 跳绳计数器

一个在浏览器本地运行的跳绳计数工具。上传视频或打开摄像头，姿态模型会追踪髋部和双脚的上下运动，估算完整跳跃次数。

## 在线使用

在线体验：<https://mentran.github.io/jump-rope-counter/>

无需安装应用，首次使用需要联网加载 TensorFlow.js 和 MoveNet 姿态模型。

## 功能

- **视频分析**：读取完整视频后统一分析动作序列。
- **实时播放计数**：模型加载完成后开始播放，并在确认一次完整起跳时立即加 1。
- **摄像头计数**：直接使用摄像头进行实时识别。
- **三种策略**：平衡、敏感、保守，用于调整漏计与误计的取舍。
- **同视频多次记录**：同一视频可以保存不同方式、不同策略的结果。
- **手动校正**：结果确认后可用 `+ / −` 做最后调整。

## 本地运行

```bash
python3 -m http.server 8000
```

然后打开 <http://localhost:8000/>，选择视频并开始计数。

## 计数原理

计数的基本单位是一次完整的上下运动周期，而不是某一帧画面。这样可以把“人在空中”和“绳子经过脚下”等瞬时画面，收敛成一个可解释的动作事件。

1. **提取姿态**：MoveNet 识别双髋、双脚的位置和置信度。
2. **构造信号**：将髋部和脚部的垂直位置归一化，得到身体上下运动信号，并做轻量平滑。
3. **视频分析**：对完整时间序列寻找局部最低点，比较两侧动作幅度，并用最短间隔去除重复候选点。
4. **实时计数**：按真实视频时间在线追踪“上升 → 下降”的方向变化；只有动作幅度和最短周期都达标时才加 1，已经确认的计数不会回退。

实时模式和视频分析是两条独立流程。实时模式只使用在线采样，不会在播放结束后再次分析视频；模型加载阶段不会播放视频，也不会计数。

## 调用的库与版本

项目没有 npm 构建步骤，页面直接通过 CDN 加载两个前端库：

- [TensorFlow.js 4.22.0](https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@4.22.0/dist/tf.min.js)：在浏览器中运行张量计算和姿态推理。
- [@tensorflow-models/pose-detection 2.1.3](https://cdn.jsdelivr.net/npm/@tensorflow-models/pose-detection@2.1.3/dist/pose-detection.min.js)：提供姿态检测接口。
- **MoveNet SinglePose Lightning**：通过 `pose-detection` 创建，用于单人实时姿态识别。

页面使用浏览器原生的 File API、`<video>`、Canvas、`requestAnimationFrame`、LocalStorage 和 WebRTC 摄像头接口，没有后端服务，也没有上传视频的接口。

## 适用范围

- 适合单人、固定机位、全身可见的视频。
- 自动结果是估计值，遮挡、快速镜头移动、多人同框都会影响结果。
- 视频帧和计数结果默认只在当前浏览器中处理和保存，不会上传到服务端。

## 项目结构

```text
index.html                       主页面与全部前端逻辑
.github/workflows/deploy-pages.yml  GitHub Pages 自动部署
```

## 技术栈

- HTML / CSS / JavaScript
- TensorFlow.js
- MoveNet SinglePose Lightning
- GitHub Pages
