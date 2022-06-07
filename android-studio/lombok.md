###  Lombok插件无效处理

Lombok插件在升级Android Studio到2021.1.1等版本后，无法正常显示，到处报红（打包正常）。

github上找到处理方式
https://github.com/mplushnikov/lombok-intellij-plugin/issues/1134

lombok 无法正常编译处理。

将lombok-plugin.zip解压后，放到你的Android studio plugins目录下即可

如：H:\Program Files\Android2020.3.1\plugins\lombok-plugin\lib

如下图方式，重启Android studio后应该就可以恢复。

如果还是行，在Android studio 设置 插件中先禁用Lombok插件，再重复上述步骤

![image](https://github.com/edmond-biguys/beja-coder/blob/main/image/85B5D809-8E93-43b7-A498-B0B19204F5E3.png)

