
# compose 使用
完整资料请参考官方文档 https://developer.android.com/jetpack/compose/documentation

## 避坑指南
1. 如果出现无法preview情况，请首先尝试rebuild，还是不行情况下，请重启Android Studio，如果还是不行，请确认kotlin版本、compose版本是否匹配。
我在开发过程中遇到kotlin版本是1.5.21，compose版本是1.0.3两个版本不匹配导致无法preview情况。

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


4. about List



