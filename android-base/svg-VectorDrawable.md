### 关于SVG
可缩放矢量图形（Scalable Vector Graphics，SVG），是一种用于描述二维的矢量图形，基于 XML 的标记语言。作为一个基于文本的开放网络标准，SVG 能够优雅而简洁地渲染不同大小的图形，并和CSS，DOM，JavaScript和SMIL等其他网络标准无缝衔接。本质上，SVG 相对于图像，就好比 HTML 相对于文本。

SVG 图像及其相关行为被定义于 XML 文本文件之中，这意味着可以对它们进行搜索、索引、编写脚本以及压缩。此外，这也意味着可以使用任何文本编辑器和绘图软件来创建和编辑它们。

和传统的点阵图像模式，像JPEG和PNG不同，SVG 格式提供的是矢量图，这意味着它的图像能够被无限放大而不失真或降低质量，并且可以方便地修改内容。

SVG 是由万维网联盟（W3C）自 1999 年开始开发的开放标准。  

[Vector Asset Studio](https://developer.android.com/studio/write/vector-asset-studio?hl=zh-cn)

[Material图标](https://fonts.google.com/icons?selected=Material+Icons&icon.style=Filled)

[阿里图标库](https://iconfont.cn/)

> M = moveto(M X,Y) ：将画笔移动到指定的坐标位置，相当于 android Path 里的moveTo()  
L = lineto(L X,Y) ：画直线到指定的坐标位置，相当于 android Path 里的lineTo()  
H = horizontal lineto(H X)：画水平线到指定的X坐标位置  
V = vertical lineto(V Y)：画垂直线到指定的Y坐标位置  
C = curveto(C X1,Y1,X2,Y2,ENDX,ENDY)：三次贝赛曲线  
S = smooth curveto(S X2,Y2,ENDX,ENDY) 同样三次贝塞尔曲线，更平滑  
Q = quadratic Belzier curve(Q X,Y,ENDX,ENDY)：二次贝赛曲线  
T = smooth quadratic Belzier curveto(T ENDX,ENDY)：映射 同样二次贝塞尔曲线，更平滑  
A = elliptical Arc(A RX,RY,XROTATION,FLAG1,FLAG2,X,Y)：弧线 ，相当于arcTo()  
Z = closepath()：关闭路径（会自动绘制链接起点和终点）
