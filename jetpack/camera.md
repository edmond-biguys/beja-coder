
[Camera官方文档](https://developer.android.com/training/camera/choose-camera-library)

在 Android 平台上，Camera、Camera2 和 CameraX 是用于访问设备相机功能的不同 API。

Camera API：Camera 是最早引入的相机 API，它提供了对设备相机的基本控制和功能访问。它使用了一种传统的相机架构，通过 Camera 类来操作相机硬件。这个 API 在 Android 5.0（API 级别 21）之前被引入，并在后续版本中逐渐被 Camera2 替代。

Camera2 API：Camera2 是 Android 5.0（API 级别 21）引入的新一代相机 API，提供了更强大和灵活的相机控制功能。它采用了基于异步回调的架构，可以更好地支持高性能相机应用程序和更精细的控制。Camera2 API 提供了更多的功能和选项，例如手动对焦、手动曝光、RAW 图片捕获等。相对于 Camera API，Camera2 API 在功能和性能上有显著的改进。

CameraX API：CameraX 是在 Android Jetpack 组件库中引入的相机 API，旨在简化相机应用程序的开发。它提供了一个更高级别的接口，使开发者能够轻松实现常见的相机功能，而不需要处理底层的设备兼容性问题。CameraX 抽象了不同设备之间的差异，提供了一致的 API 接口，使开发者能够更轻松地编写跨设备的相机应用程序。CameraX 还提供了对摄像头生命周期、图像分析、图像捕获等常见任务的支持。Google推荐使用CameraX，```For most developers, we recommend CameraX```

总结来说，Camera 是旧的相机 API，Camera2 是新一代相机 API，提供更高级的功能和性能。而 CameraX 是基于 Jetpack 组件库的相机 API，旨在简化相机应用程序的开发，并提供跨设备的兼容性。使用 CameraX 可以更方便地编写适配不同设备的相机应用程序。

简单来说，对于普通开发者而言，99%的情况下，Google推荐使用[CameraX](https://developer.android.com/training/camerax?hl=zh-cn)。

以下我们来记录下CameraX的使用基本方法、常见API及更进一步的了解关于CameraX，Google替我们做了哪些。

设备兼容  
CameraX 支持搭载 Android 5.0（API 级别 21）或更高版本的设备，覆盖现有 Android 设备的 98% 以上。

易用性  
预览：在屏幕上查看图片。  
图片分析：无缝访问缓冲区中的图片以便在算法中使用，例如将其传递到机器学习套件。  
图片拍摄：保存图片。  
视频拍摄：保存视频和音频。  

确保各设备间的一致性  
要维持一致的相机行为并非易事。您必须考虑宽高比、屏幕方向、旋转角度、预览大小和图像大小。有了 CameraX，这些基本行为都不用您再费心。  
我们设立了一个自动化 CameraX 测试实验室（如下图），用于测试搭载 Android 5.0 及更高版本的一系列设备和这些操作系统版本中的各种相机行为。我们将持续运行这些测试，以找出各种各样的问题并进行修复。  
<img src="https://developer.android.com/static/images/training/camera/camerax/testing-lab.png?hl=zh-cn" width=400>

相机扩展，CameraX有一个extensions API，扩展程序包含焦外成像（人像）、高动态范围 (HDR)、夜间模式和脸部照片修复功能，所有这些都需要设备支持。

CameraX架构 （CameraX architecture）
