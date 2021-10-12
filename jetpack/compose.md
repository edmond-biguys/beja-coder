
# compose 使用
完整资料请参考官方文档 https://developer.android.com/jetpack/compose/documentation

## 避坑指南
1. 如果出现无法preview情况，请首先尝试rebuild，还是不行情况下，请重启Android Studio，如果还是不行，请确认kotlin版本、compose版本是否匹配。
我在开发过程中遇到kotlin版本是1.5.21，compose版本是1.0.3两个版本不匹配导致无法preview情况。

2. 没有start interactive mode按钮处理方法，请在Android Studio设置中，搜索enable interactive，找到 “Enable interactive and animation preview tools”并勾选。


compose 可以预览动画。
