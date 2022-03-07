查看视频信息

```ffmpeg -i xxx.mp4```

视频转换分辨率
```ffmpeg -i behavior01.mp4 -vf scale=720:1280 behavior01_720p.mp4 ```

视频字幕提取
```ffmpeg -i chase.mkv -map 0:s:0 subs.srt```
-i 表示输入文件

-map 表示视频中的那部分

0:s:0 第一个0表示第一个视频

0:s:0 s表示是字幕

0:s:0 第二个0表示第几个字幕文件

subs.srt表示输出文件名称
