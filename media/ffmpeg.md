## ffmpeg使用记录

### ffmpeg安装（mac）

如果没有安装homebrew 请先安装homebrew工具。

然后安装ffmpeg，其实只要一个命令：```brew install ffmpeg```, 安装完成后，使用```brew info ffmpeg```来查看安装结果。

理论上，这应该就已经安装结束了，实际上我遇到安装失败了，原因很奇怪，最后也没有找到失败原因，但是通过另一种方法安装成功了，失败显示如下，

```
curl: (22) The requested URL returned error: 404 
Warning: Bottle missing, falling back to the default domain...
==> Downloading https://ghcr.io/v2/homebrew/core/ffmpeg/manifests/4.4_2
Already downloaded: /Users/caoj/Library/Caches/Homebrew/downloads/7e46d07e58558787f2462db0478b67d0177fb5c12e0556c2f4974e7177343217--ffmpeg-4.4_2.bottle_manifest.json
==> Downloading https://ghcr.io/v2/homebrew/core/ffmpeg/blobs/sha256:9da28933b9f1abc3b1cf92382d1a8ea051c98f9dd0f4ef47e8d37d2aa9a4769a
Already downloaded: /Users/caoj/Library/Caches/Homebrew/downloads/1a68ea3249dfabb73c20a3c924ade30fa4f9adb0efca201d929b37db03fbc652--ffmpeg--4.4_2.big_sur.bottle.tar.gz
==> Installing dependencies for ffmpeg: pcre, glib, libpthread-stubs, xorgproto, libxau, libxdmcp, libxcb, libx11, libxext, libxrender, lzo, pixman, cairo, gobject-introspection, graphite2, icu4c, harfbuzz, libass, libbluray, libsoxr, libvidstab, libogg, libvorbis, libvpx, opencore-amr, jpeg, libtiff, little-cms2, openjpeg, opus, rav1e, flac, libsndfile, libsamplerate, rubberband, sdl2, snappy, speex, srt, giflib, webp, leptonica, tesseract, theora, x264, x265, xvid, libsodium, zeromq and zimg
==> Installing ffmpeg dependency: pcre
==> Pouring pcre-8.45.big_sur.bottle.tar.gz
🍺  /usr/local/Cellar/pcre/8.45: 204 files, 5.8MB
==> Installing ffmpeg dependency: glib
==> Pouring glib-2.70.0.big_sur.bottle.tar.gz
Error: No such file or directory @ rb_sysopen - /Users/caoj/Library/Caches/Homebrew/downloads/8a904d00313d86aa0a019b7db649170254cd77781165d4b2b86aafeadcafe09b--glib-2.70.0.big_sur.bottle.tar.gz
```
提示是在安装```glib```时，找不到下载的文件，，但实际是有这个文件的，只是自动下载后的文件和安装解压时的文件名不一样，然后程序就找不到文件了，不清楚为什么这两个文件名会不一样。
既然不能自动安装，那么智能手动把需要安装的包都安装下，当然是程序员的手动方式，使用如下shell脚本进行下载，

需要下载的包的列表
> aom, dav1d, libpng, freetype, fontconfig, frei0r, gmp, bdw-gc, libffi, m4, libtool, libunistring, pkg-config, guile, gettext, libidn2, libtasn1, nettle, p11-kit, libevent, c-ares, jemalloc, libev, nghttp2, unbound, gnutls, lame, fribidi, 
pcre, glib, libpthread-stubs, xorgproto, libxau, libxdmcp, libxcb, libx11, libxext, libxrender, lzo, pixman, cairo, gobject-introspection, graphite2, icu4c, harfbuzz, libass, libbluray, libsoxr, libvidstab, libogg, libvorbis, libvpx, opencore-amr, jpeg, libtiff, little-cms2, openjpeg, opus, rav1e, flac, libsndfile, libsamplerate, rubberband, sdl2, snappy, speex, srt, giflib, webp, leptonica, tesseract, theora, x264, x265, xvid, libsodium, zeromq and zimg

单独下载某一个的命令如下

```brew install aom```


那么接下来，我们就开始批量下载，新建一个shell脚本文件

```touch packageInstall.sh```

使用su权限，输入用户密码

```sudo su```

给packageInstall.sh文件权限

```chmod 755 packageInstall.sh```

退出su

```exit```

将如下指令添加到packageInstall.sh文件中，使用```./packageInstall.sh```指令执行这个shell脚本，接下来等待指令执行完成。
```
#!/bin/bash
# This is our ffmpeg install script


# my_array=('libxcb' 'libx11' 'libxext')
my_array=('aom' 'dav1d' 'libpng' 'freetype' 'fontconfig' 'frei0r' 'gmp' 'bdw-gc' 'libffi' 'm4' 'libtool' 'libunistring' 'pkg-config' 'guile' 'gettext' 'libidn2' 'libtasn1' 'nettle' 'p11-kit' 'libevent' 'c-ares' 'jemalloc' 'libev' 'nghttp2' 'unbound' 'gnutls' 'lame' 'fribidi' 
'pcre' 'glib' 'libpthread-stubs' 'xorgproto' 'libxau' 'libxdmcp' 'libxcb' 'libx11' 'libxext' 'libxrender' 'lzo' 'pixman' 'cairo' 'gobject-introspection' 'graphite2' 'icu4c' 'harfbuzz' 'libass' 'libbluray' 'libsoxr' 'libvidstab' 'libogg' 'libvorbis' 'libvpx' 'opencore-amr' 'jpeg' 'libtiff' 'little-cms2' 'openjpeg' 'opus' 'rav1e' 'flac' 'libsndfile' 'libsamplerate' 'rubberband' 'sdl2' 'snappy' 'speex' 'srt' 'giflib' 'webp' 'leptonica' 'tesseract' 'theora' 'x264' 'x265' 'xvid' 'libsodium' 'zeromq' 'zimg')

echo "my_array size ${my_array[@]}"
echo "my_array size ${#my_array[*]}"

for var in ${my_array[*]}
do 
    echo "---------> start install $var !!!!!!"
    brew install $var
    echo "---------> install $var end <-----------"
done
```

上述指令执行完成后，再次执行```brew install ffmpeg```，然后看通过```brew info ffmpeg```查看下是否安装成功，也可以通过ffmpeg指令来测试验证下，是否真的可以使用了，
比如将视频文件转换成gif图片```ffmpeg -i xxx.mp4 xxx.gif```。

好了，以上就是我mac上安装ffmpeg遇到的问题。













