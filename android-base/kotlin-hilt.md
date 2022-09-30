### Hilt

[官方文档](https://developer.android.com/training/dependency-injection/hilt-android)

#### 添加依赖

1. 在工程root下的build.gradle中添加plugin

```gradle
buildscript {
    ...
    dependencies {
        ...
        classpath("com.google.dagger:hilt-android-gradle-plugin:2.38.1")
    }
}
```

2. 在app的build.gradle中应用plugin，并添加依赖

   ```gradle
   plugins {
       //使用kotlin("kapt")这个时，会报这个错误 only id(String) method calls allowed in plugins {}，修改为下边的内容，就可以了
       //kotlin("kapt") 为kotlin的写法，不是groovy语法
       id("kotlin-kapt")
       id("dagger.hilt.android.plugin")
   }

   android {
       ...
   }

   dependencies {
       implementation("com.google.dagger:hilt-android:2.38.1")
       kapt("com.google.dagger:hilt-android-compiler:2.38.1")
   }

   // Allow references to generated code
   kapt {
    correctErrorTypes = true
   }
   ```
3. hilt使用java8功能，在app的build.gradle中添加配置

   ```gradle
   android {
       ...
       compileOptions {
           sourceCompatibility = JavaVersion.VERSION_1_8
           targetCompatibility = JavaVersion.VERSION_1_8
       }
   }
   ```


#### 在应用中使用Hilt

所有使用Hilt的应用，都必须包含一个带有@HiltAndroidApp注解的Application。  

@HiltAndroidApp会触发Hilt的代码生成操作，生成的代码包括应用的一个基类，该基类充当应用级依赖项容器。

在 `Application` 类中设置了 Hilt 且有了应用级组件后，Hilt 可以为带有 `@AndroidEntryPoint` 注释的其他 Android 类提供依赖项：

Hilt目前支持的Android类有，Application(通过使用HiltAndroidApp)，ViewModel(通过HiltViewModel)，Activity, Fragment, View, Service, BroadcastReceiver。

如果您使用 `@AndroidEntryPoint` 为某个 Android 类添加注释，则还必须为依赖于该类的 Android 类添加注释。例如，如果您为某个 fragment 添加注解，则还必须为使用该 fragment 的所有 activity 添加注解。


 **注意** ：在 Hilt 对 Android 类的支持方面适用以下几项例外情况：

*  Hilt 仅支持扩展 [`ComponentActivity`](https://developer.android.com/reference/kotlin/androidx/activity/ComponentActivity) 的 activity，如 [`AppCompatActivity`](https://developer.android.com/reference/kotlin/androidx/appcompat/app/AppCompatActivity)。

* Hilt 仅支持扩展 `androidx.Fragment` 的 Fragment。
* Hilt 不支持保留的 fragment。

`@AndroidEntryPoint` 会为项目中的每个 Android 类生成一个单独的 Hilt 组件。这些组件可以从它们各自的父类接收依赖项。

如需从组件获取依赖项，请使用 `@Inject` 注释执行字段注入。
