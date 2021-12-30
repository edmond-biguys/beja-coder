## 如何优雅的让其他开发者使用我们的库。
我们经常通过gradle导入第三方库，那么我们自己的库如何提供给其他开发者使用？下边我们分别介绍使用jitpack、mavenCentral（以前叫JCenter）来发布我们的库。
在使用前，我们先了解下JCenter、jitpack、mavenCentral为何物。

## JCenter
JCenter是JFrog公司提供的Android第三方库的仓库，JFrog公司宣布即将废弃该仓库，jcenter仓库是也曾经google默认推荐的第三方库。最早宣布废弃时，2022年2月后，将不可以下载上边的库，如果这些库的开发
者不做库迁移，那么普通开发者将无法使用这些库，不过，好在最后JFrog公司可能和Google达成了什么协议，后续还能下载，但不能更新维护。目前Google推荐使用mavenCentral仓库。

## mavenCentral
sonatype公司提供的第三方仓库，当时使用比较麻烦，审核也比较严格，比如你发布库的时候，库的包名，你必须要有这个域名的所有权，才能发布，不像jcenter谁先用，就归谁。目前Google推荐使用的第三方仓库。

## jitpack (https://jitpack.io/)
在jcenter废弃后，逐渐被用的越来越多，使用比较简单，适合个人开发者使用，缺点是不是Google官方推荐，使用时要手动添加maven依赖。
这里我们主要讲下jitpack使用，原因是使用简单，另外网上大部分资料都是使用这个第三方插件```classpath 'com.github.dcendents:android-maven-gradle-plugin:2.1'```，这里不推荐使用，建议使用
官方提供上传方法，原因是第三方库为个人维护，更新比较不稳定，不支持高版本gradle插件。

官方使用方法如下，
#### 1. 新建library Module
在你当前的项目下，新建一个library的Module工程，该module工程就是你要发布的library库，如下图
<img src="https://github.com/edmond-biguys/beja-coder/blob/main/jetpack/image/compose-view.png" width="300px">
在该module工程的build.gradle配置文件下，增加如下内容，
```
plugins {
    ...
    id 'maven-publish'
}
group = 'com.github.edmond-biguys'
version = '1.0'

afterEvaluate {
    publishing {
        publications {
            // Creates a Maven publication called "release".
            release(MavenPublication) {
                from components.release
                groupId = 'com.github.edmond-biguys'
                artifactId = 'android-compose-view'
                version = '1.0'
            }
        }
    }
}

dependencies {
    ...
}
```
你也可以查看gitpack官方提供的build.gradle文件的编写，https://github.com/jitpack/android-example/blob/master/library/build.gradle
到此，我们就完成了所有的工程配置，简单吗？

#### 2. 上传library工程，创建release版本
如何push工程到github请自己百度，在使用jitpack生成依赖库前，你需要先给你的工程打包一个release版本，方法如下图
<img src="https://github.com/edmond-biguys/beja-coder/blob/main/jetpack/image/github-create-tag01.png" width="500px">

<img src="https://github.com/edmond-biguys/beja-coder/blob/main/jetpack/image/github-create-tag02.png" width="500px">

到此，你已完成了github工程，release版本创建，接下来该在jitpack上打包依赖库了。

#### 3. 在jitpack上编译该release版本
将你的项目git地址（如我的项目地址是https://github.com/edmond-biguys/Android-base-sample ）输入到jitpack，如下图

<img src="https://github.com/edmond-biguys/beja-coder/blob/main/jetpack/image/jitpack-build01.png" width="600px">

输入你的项目地址后，点击Look up按钮，就可以看到你刚才打的release版本，点击Get it按钮，jitpack就会开始编译，编译成功结果会显示绿色，失败则是红色，如下图

<img src="https://github.com/edmond-biguys/beja-coder/blob/main/jetpack/image/jitpack-build02.png" width="500px">

上图中0.0.1-alpha05版本编译失败了，结果显示为红色，点击结果进去后，可以看到失败原因，我们可以根据失败原因进行修复。
上图中v0.0.1-alpha06版本编译成功了，结果显示为绿色，点击结果进去后，可以看到编译结果，如下内容，
```
BUILD SUCCESSFUL in 5m 27s
289 actionable tasks: 283 executed, 6 up-to-date
Build tool exit code: 0
Looking for artifacts...
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2
Looking for pom.xml in build directory and ~/.m2
[Fatal Error] lint-resources-release.xml:1:1: Content is not allowed in prolog.
Found artifact: com.github.edmond-biguys:android-compose-view:1.0
2021-12-29T09:33:47.832646753Z
Exit code: 0

Build artifacts:
com.github.edmond-biguys:Android-base-sample:v0.0.1-alpha06
```
从上述结果可以看到，BUILD SUCCESSFUL，编译时间有点久，5m 27s，同时可以看到关键内容，我们的依赖库 ```com.github.edmond-biguys:Android-base-sample:v0.0.1-alpha06```
到此，我们已经完成了依赖库的编译，并得到依赖库的地址，接下来我们看下如何使用我们的依赖库。

#### 4. 使用你的依赖库

依赖库使用比较简单，在工程的build.gradle文件下，增加maven依赖，
```
  repositories {
        maven { url 'https://jitpack.io' }
  }
allprojects {
    repositories {
        maven { url 'https://jitpack.io' }
    }
}
```
在app的build.gradle中增加依赖，
```
dependencies {
    implementation("com.github.edmond-biguys:Android-base-sample:v0.0.1-alpha06")
}
```
接下来就可以正常使用这个依赖库了。

