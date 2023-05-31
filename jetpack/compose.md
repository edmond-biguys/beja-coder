
# compose 使用
完整资料请参考官方文档 https://developer.android.com/jetpack/compose/documentation

Google官方提供Jetpack Compose教程 https://developer.android.google.cn/courses/pathways/compose?hl=zh_cn

## 快速接入
1. 使用最新版本Android Studio
2. 在app的build.gradle文件中增加相应依赖，及启用信息

```
android {
...
    buildFeatures {
        // Enables Jetpack Compose for this module
        compose true
    }
    composeOptions {
        kotlinCompilerExtensionVersion '1.0.5'
    }
...

}

dependencies {
    implementation("androidx.activity:activity-compose:1.4.0")
    implementation("androidx.compose.ui:ui:1.0.5")
    // Tooling support (Previews, etc.)
    implementation("androidx.compose.ui:ui-tooling:1.0.5")
    // Foundation (Border, Background, Box, Image, Scroll, shapes, animations, etc.)
    implementation("androidx.compose.foundation:foundation:1.0.5")
    // Material Design
    implementation("androidx.compose.material:material:1.0.5")
    // Material design icons
    implementation("androidx.compose.material:material-icons-core:1.0.5")
    implementation("androidx.compose.material:material-icons-extended:1.0.5")
    // Integration with observables
    implementation("androidx.compose.runtime:runtime-livedata:1.0.5")
    implementation("androidx.compose.runtime:runtime-rxjava2:1.0.5")
    
    //类似与以前的support库
    api "com.google.accompanist:accompanist-insets:0.21.3-beta"
    api "com.google.accompanist:accompanist-insets-ui:0.21.3-beta"
    api "com.google.accompanist:accompanist-systemuicontroller:0.21.3-beta"

    // UI Tests
    androidTestImplementation("androidx.compose.ui:ui-test-junit4:1.0.5")
}
```

## 避坑指南
1. 如果出现无法preview情况，请首先尝试rebuild，还是不行情况下，请重启Android Studio，如果还是不行，请确认kotlin版本、compose版本是否匹配。
我在开发过程中遇到kotlin版本是1.5.21，compose版本是1.0.3两个版本不匹配导致无法preview情况。但，所有的库，请依然使用当前的最新版本，在兼容性有问题时候，再去尝试使用老版本，请跟上时代，不要被时代抛弃。  
这里补充一点，[kotlin版本和compose版本对应版本查找](https://developer.android.com/jetpack/androidx/releases/compose-kotlin)，官方有对应查找表。

2. 没有start interactive mode按钮处理方法，请在Android Studio设置中，搜索enable interactive，找到 “Enable interactive and animation preview tools”并勾选。


compose 可以预览动画。

## compose 使用记录

1. 关于constraintLayout，在以前的View System UI中，constraintLayout用来减少嵌套，more nested means more measurement，that means low performance，并且嵌套越多，计算量是指数级增加的，所以推荐使用constraint layout。但是在compose中，是否选择使用constraint layout，只在他是否对代码可读性、维护性有帮助。
    
   >  but in compose, it's not the same, whether to use ConstraintLayout or not for a particular UI in compose is up to the developer's preference. In the Android View  system, ConstraintLayout was used as a way to build more performant layouts, but this is not a concern in compose. When needing to choose, consider if 
   >  ConstraintLayout helps with the readability and maintainability of the composable.
    
2. 在compose中，子控件只能被测量一次，如果测量两次，就会报运行时错误。但是在compose中，会提供一些固有的属性，当多次读取这些属性时，不会进行测量，并不会报错。
> One of the rules of compose is that you should only measure your children once; measuring children twice throws a runtime exception. However, there are times when you need some information about your children before measuring them.
> Intrinsic lets you query children before they're actually measured.

3. about Text()
在使用Text时，出现一个问题，Text中的alignment参数只能控制水平方向，无法控制垂直方向，官方文档也有说明这一点，text中的alignmnent和布局中的alignment是不一样的，并且官方关于text中alignment的说明，只提到了水平，完全不说垂直方案，从侧面也反应不支持垂直方向控制。如果需要控制垂直方案，请在外部套一层布局，如Box，再对Box中的Text进行布局控制。
> Note: Text alignment is different from layout alignment, which is about positioning a Composable within a container such as a Row or Column. Check out the Compose layout basics documentation to read more about it.


4. about List控件
*  使用List控件时，如果从业务上已经明确，这个list是不可变的，或者内容不会太多，那么可以考虑简单的使用Column控件，如果需要滚动，可以通过verticalScroll来控制，如下简单代码，
```
@Composable
fun MessageList(messages: List<String>) {
    val rememberState = rememberScrollState()
    Column(
        modifier = Modifier
            .verticalScroll(state = rememberState)
            .fillMaxWidth()
        ) {
        messages.forEachIndexed { index, s ->
            var color = blue60
            if (index % 2 == 0) {
               color = blue20
            }
            Box(
                modifier = Modifier
                    .fillMaxSize()
                    .height(50.dp)
                    .background(color),
                contentAlignment = Alignment.Center
            ) {
                Text(
                    modifier = Modifier
                        .align(alignment = Alignment.Center),
                    text = s,
                    textAlign = TextAlign.Center

                )
            }
        }
    }
}
```

* 但是当你要展示的列表item比较多，或者不确定item数量时，再使用Column就不合适了，因为使用Column时，不管是否在屏幕上有显示，都会占用内存，会导致性能问题，这时，我们就需要使用到LazyColumn和LazyRow，这两个控件和RecyclerView有相同的规范。使用方法如下，
```
//if the list is large or the items is unknown, you can use the LazyColumn or LazyRow
@Composable
fun MessageListForLazyColumn(messages: List<String>) {
    LazyColumn(
        contentPadding = PaddingValues(horizontal = 16.dp, vertical = 8.dp),
        modifier = Modifier.background(blue20).fillMaxHeight(),
        verticalArrangement = Arrangement.SpaceAround) {
        //this is items
        items(messages.size) { index ->
            val color = if (index % 2 == 0) blue20 else blue60
            Box(modifier = Modifier
                .fillMaxWidth()
                .height(50.dp)
                .background(color),
            contentAlignment = Alignment.Center,
                ) {
                Text(text = messages[index])
            }
        }
    }
}
```
这里特别需要注意，verticalArrangement = Arrangement.SpaceAround在使用时，包括Arrangement下的其他一些方法、参数，只有在item之间生效，第一次写的时候，我把所有内容写到了一个item中，所以一直不生效，费了一些时间，请注意下边的写法，
```
//正确的写法
itemsIndexed(messages) { index, string -> 
    ...
}
```

```
//正确的写法
messages.forEachIndexed { index, string -> 
    item {
        ...
    }
}
```

```
//错误的写法
//you will make a huge item.
item {
    messages.forEachIndexed { index, string ->
        Box {
            Text()
            ...
        }
    }
}
```




