
[Camera官方文档](https://developer.android.com/training/camera/choose-camera-library)

在 Android 平台上，Camera、Camera2 和 CameraX 是用于访问设备相机功能的不同 API。

Camera API：Camera 是最早引入的相机 API，它提供了对设备相机的基本控制和功能访问。它使用了一种传统的相机架构，通过 Camera 类来操作相机硬件。这个 API 在 Android 5.0（API 级别 21）之前被引入，并在后续版本中逐渐被 Camera2 替代。

Camera2 API：Camera2 是 Android 5.0（API 级别 21）引入的新一代相机 API，提供了更强大和灵活的相机控制功能。它采用了基于异步回调的架构，可以更好地支持高性能相机应用程序和更精细的控制。Camera2 API 提供了更多的功能和选项，例如手动对焦、手动曝光、RAW 图片捕获等。相对于 Camera API，Camera2 API 在功能和性能上有显著的改进。

CameraX API：CameraX 是在 Android Jetpack 组件库中引入的相机 API，旨在简化相机应用程序的开发。它提供了一个更高级别的接口，使开发者能够轻松实现常见的相机功能，而不需要处理底层的设备兼容性问题。CameraX 抽象了不同设备之间的差异，提供了一致的 API 接口，使开发者能够更轻松地编写跨设备的相机应用程序。CameraX 还提供了对摄像头生命周期、图像分析、图像捕获等常见任务的支持。Google推荐使用CameraX，```For most developers, we recommend CameraX```

总结来说，Camera 是旧的相机 API，Camera2 是新一代相机 API，提供更高级的功能和性能。而 CameraX 是基于 Jetpack 组件库的相机 API，旨在简化相机应用程序的开发，并提供跨设备的兼容性。使用 CameraX 可以更方便地编写适配不同设备的相机应用程序。

简单来说，对于普通开发者而言，99%的情况下，Google推荐使用[CameraX](https://developer.android.com/training/camerax?hl=zh-cn)。

以下我们来记录下CameraX的使用基本方法、常见API及更进一步的了解关于CameraX，Google替我们做了哪些。

### 设备兼容  
CameraX 支持搭载 Android 5.0（API 级别 21）或更高版本的设备，覆盖现有 Android 设备的 98% 以上。

### 易用性  
* 预览：在屏幕上查看图片。  
* 图片分析：无缝访问缓冲区中的图片以便在算法中使用，例如将其传递到机器学习套件。  
* 图片拍摄：保存图片。  
* 视频拍摄：保存视频和音频。  

### 确保各设备间的一致性  
要维持一致的相机行为并非易事。您必须考虑宽高比、屏幕方向、旋转角度、预览大小和图像大小。有了 CameraX，这些基本行为都不用您再费心。  
我们设立了一个自动化 CameraX 测试实验室（如下图），用于测试搭载 Android 5.0 及更高版本的一系列设备和这些操作系统版本中的各种相机行为。我们将持续运行这些测试，以找出各种各样的问题并进行修复。  
<img src="https://developer.android.com/static/images/training/camera/camerax/testing-lab.png?hl=zh-cn" width=400>

### 相机扩展  
CameraX有一个extensions API，扩展程序包含焦外成像（人像）、高动态范围 (HDR)、夜间模式和脸部照片修复功能，所有这些都需要设备支持。

### CameraX架构 （CameraX architecture）  
架构包含了结构（structure），API使用，生命周期（lifecycles），用例如何组合使用（how to combine use cases）

#### 结构（structure）
* 预览 Preview
* 图片分析Image analysis
* 图片拍摄 Capture Image
* 视频拍摄 Capture Video  
上述的用例，你也有组合使用，比如在图片拍摄时，使用图片分析（take a picture when the people in the photo are smiling）

#### 开始使用CameraX
##### 添加依赖
打开项目的 settings.gradle 文件并添加 google() 代码库，如下所示：  
```
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
```
在build.gradle中添加依赖  
```
// CameraX core library using the camera2 implementation
    def camerax_version = "1.3.0-alpha07"
    // The following line is optional, as the core library is included indirectly by camera-camera2
    implementation "androidx.camera:camera-core:${camerax_version}"
    implementation "androidx.camera:camera-camera2:${camerax_version}"
    // If you want to additionally use the CameraX Lifecycle library
    implementation "androidx.camera:camera-lifecycle:${camerax_version}"
    // If you want to additionally use the CameraX VideoCapture library
    implementation "androidx.camera:camera-video:${camerax_version}"
    // If you want to additionally use the CameraX View class
    implementation "androidx.camera:camera-view:${camerax_version}"
    // If you want to additionally add CameraX ML Kit Vision Integration
    implementation "androidx.camera:camera-mlkit-vision:${camerax_version}"
    // If you want to additionally use the CameraX Extensions library
    implementation "androidx.camera:camera-extensions:${camerax_version}"
```  

#### 预览Preview

然后在布局文件xml中添加一个PreviewView，代码如下
```
<androidx.camera.view.PreviewView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="center"
        android:id="@+id/previewView"
        />
```
PreviewView是一个自定义的view,用来显示camera的数据，PreviewView默认使用SurfaceView，当让你也可以通过设置来修改为TextureView，设置方法如下，
```
//这个是默认模式，使用surfaceView
previewView.implementationMode = PreviewView.ImplementationMode.PERFORMANCE

previewView.implementationMode = PreviewView.ImplementationMode.COMPATIBLE
```

针对上边的使用以下是官方说明
>Use a SurfaceView for the preview when possible. If the device doesn't support SurfaceView, PreviewView will fall back to use a TextureView instead.  
PreviewView falls back to TextureView when the API level is 24 or lower, the camera hardware support level is CameraCharacteristics.INFO_SUPPORTED_HARDWARE_LEVEL_LEGACY, or Preview.getTargetRotation() is different from PreviewView's display rotation.  
Do not use this mode if Preview.Builder.setTargetRotation(int) is set to a value different than the display's rotation, because SurfaceView does not support arbitrary rotations. Do not use this mode if the PreviewView needs to be animated. SurfaceView animation is not supported on API level 24 or lower. Also, for the preview's streaming state provided in getPreviewStreamState, the PreviewView.StreamState.STREAMING state might happen prematurely if this mode is used.  
See Also:  
Preview.Builder.setTargetRotation(int),  
Preview.Builder.getTargetRotation(),  
Display.getRotation(),  
CameraCharacteristics.INFO_SUPPORTED_HARDWARE_LEVEL_LEGACY,  
PreviewView.StreamState.STREAMING  

默认使用SurfaceView, 在不支持的设备上（如API小于等于24的），会降级使用TextureView  
相机预览的使用方法如下
```
    private lateinit var cameraProviderFuture : ListenableFuture<ProcessCameraProvider>

    private fun initPreview() {
        cameraProviderFuture = ProcessCameraProvider.getInstance(requireActivity())

        cameraProviderFuture.addListener({
            val cameraProvider = cameraProviderFuture.get()
            bindPreview(cameraProvider)
        }, ContextCompat.getMainExecutor(requireActivity()))
    }

    private fun bindPreview(cameraProvider: ProcessCameraProvider) {
        val preview = Preview.Builder().build()
        val cameraSelector = CameraSelector.Builder()
            .requireLensFacing(CameraSelector.LENS_FACING_BACK)
            .build()
        //TextureViewImpl          D  Surface set on Preview.
//        binding.previewView.implementationMode = PreviewView.ImplementationMode.COMPATIBLE


        //SurfaceViewImpl          D  Surface set on Preview.
        //这个是默认模式
        with(binding) {
            previewView.implementationMode = PreviewView.ImplementationMode.PERFORMANCE
            previewView.scaleType = PreviewView.ScaleType.FIT_CENTER

        }
//        binding.previewView.scaleX = 2.2F
//        binding.previewView.scaleY = 2.2F
        preview.setSurfaceProvider(binding.previewView.surfaceProvider)
        //这里preview会返回一个camera对象，可以获取camera的一些信息，也可以对camera进行一些控制
        val camera = cameraProvider.bindToLifecycle(this as LifecycleOwner, cameraSelector, preview)

    }
```

#### 图片拍摄 ImageCapture
```
private var lensFacing = CameraSelector.LENS_FACING_BACK
    private fun bindCameraUseCase(cameraProvider: ProcessCameraProvider) {
        val cameraSelector = CameraSelector.Builder()
            .requireLensFacing(lensFacing)
            .build()

        preview = Preview.Builder()
            .build()
        binding.previewView.scaleType = PreviewView.ScaleType.FIT_CENTER
        preview.setSurfaceProvider(binding.previewView.surfaceProvider)


        imageCapture = ImageCapture.Builder()
            .setCaptureMode(ImageCapture.CAPTURE_MODE_MINIMIZE_LATENCY)
            .build()

        //在bind前，先unbind。比如切换摄像头等。
        cameraProvider.unbindAll()

        val camera = cameraProvider.bindToLifecycle(this, cameraSelector, preview, imageCapture)
        observeCamera(camera)
    }
```
在图片拍摄和预览其实差不多，主要就是创建相应的useCase，然后bind。  
这里的 ```val camera = cameraProvider.bindToLifecycle(this, cameraSelector, preview, imageCapture)```这个bind方法，可以传入多个useCase，理论上拍照是不需要preview画面，就可以拍摄的，但实际无画面拍的是什么都不知道。

如下是，按下拍照按钮后，执行的代码逻辑
```
private fun capture() {

        val name = SimpleDateFormat("yyyy-MM-dd-HH-mm-ss-SSS", Locale.CHINA).format(System.currentTimeMillis())

        val contentValues = ContentValues().apply {
            put(MediaStore.MediaColumns.DISPLAY_NAME, name)
            put(MediaStore.MediaColumns.MIME_TYPE, "image/jpeg")
            val appName = requireContext().resources.getString(R.string.app_name)
            put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/$appName")
        }

        val outputOptions = ImageCapture.OutputFileOptions
            .Builder(requireContext().contentResolver, MediaStore.Images.Media.EXTERNAL_CONTENT_URI, contentValues)
            .build()

        //实际上拍照，不用预览也是可以拍的，只不过没有预览，你不知道拍的是什么内容
        imageCapture.takePicture(outputOptions, cameraExecutorService,
            object : ImageCapture.OnImageSavedCallback {
                override fun onImageSaved(output: ImageCapture.OutputFileResults) {
                    val savedUri = output.savedUri
                    Log.i(TAG, "onImageSaved: $savedUri")

                    if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.M) {
                        //设置缩略图
                        setGalleryThumbnail(savedUri.toString())
                    }

                    // Implicit broadcasts will be ignored for devices running API level >= 24
                    // so if you only target API level 24+ you can remove this statement
                    if (android.os.Build.VERSION.SDK_INT < android.os.Build.VERSION_CODES.N) {
                        // Suppress deprecated Camera usage needed for API level 23 and below
                        @Suppress("DEPRECATION")
                        requireActivity().sendBroadcast(
                            Intent(android.hardware.Camera.ACTION_NEW_PICTURE, savedUri)
                        )
                    }


                }

                override fun onError(exception: ImageCaptureException) {
                    Log.i(TAG, "onError: $exception")
                }

            })
    }
    
    //拍照完成后，在跳转到相册按钮上，显示上一张照片的缩略图。
    private fun setGalleryThumbnail(fileName: String) {
        with(binding) {
            buttonThumbnail.post {
                buttonThumbnail.setPadding(resources.getDimension(R.dimen.stroke_small).toInt())

                Glide.with(buttonThumbnail)
                    .load(fileName)
                    .apply(RequestOptions.circleCropTransform())
                    .into(buttonThumbnail)
            }
        }
    }
```



















