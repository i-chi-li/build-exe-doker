# はじめに
このプロジェクトは、ffmpeg をビルドする。
ffmpeg の各実行ファイルは、DLL なしで単体起動できる。
Docker コンテナをビルドすると、ffmpeg の各実行ファイルが生成される。


# ffmpeg をビルド

何度もビルドしない場合は、以下の手順で ffmpeg 実行ファイルを生成する。

```shell
cd ffmpeg
docker compose up -d --build
```

完了までには数十分かかる。

完了すると、```/root/ffmpeg-build/ffmpeg-5.1.1``` に
ffmpeg.exe, ffplay.exe, ffprobe.exe などが生成される。

実行中に以下のようなメッセージが表示されるが問題ない。
```[output clipped, log limit 1MiB reached]```


# 複数回 ffmpeg ビルドを行う場合

利用ソースのバージョン変更や、新規ライブラリの追加などで、
試行錯誤が必要な場合は、apt パッケージのキャッシュを有効化出来る。

## apt キャッシュ用コンテナ

ffmpeg をビルドする前に、apt キャッシュ用のコンテナを起動する。

```shell
cd apt-cacher-ng
docker compose up -d --build
cd ..
```

```ffmpeg/docker/Dockerfile``` ファイルを以下のように変更する。

```ARG USE_PROXY=NO``` -> ```ARG USE_PROXY=YES```

変更後、前述の手順通りに起動する。

```shell
cd ffmpeg
docker compose build
cd ..
```


# 依存 DLL チェック

実行ファイル単体で実行できるかどうかをチェックする。

```
~/ffmpeg-build/ffmpeg-5.1.1# x86_64-w64-mingw32-objdump -p ffmpeg.exe | rg 'DLL Name'
DLL Name: ADVAPI32.dll
DLL Name: bcrypt.dll
DLL Name: CRYPT32.dll
DLL Name: GDI32.dll
DLL Name: IMM32.dll
DLL Name: KERNEL32.dll
DLL Name: msvcrt.dll
DLL Name: ole32.dll
DLL Name: OLEAUT32.dll
DLL Name: PSAPI.DLL
DLL Name: SETUPAPI.dll
DLL Name: SHELL32.dll
DLL Name: SHLWAPI.dll
DLL Name: USER32.dll
DLL Name: VERSION.dll
DLL Name: AVICAP32.dll
DLL Name: WINMM.dll
DLL Name: WS2_32.dll

~/ffmpeg-build/ffmpeg-5.1.1# x86_64-w64-mingw32-objdump -p ffplay.exe | rg 'DLL Name'
DLL Name: ADVAPI32.dll
DLL Name: bcrypt.dll
DLL Name: CRYPT32.dll
DLL Name: GDI32.dll
DLL Name: IMM32.dll
DLL Name: KERNEL32.dll
DLL Name: msvcrt.dll
DLL Name: ole32.dll
DLL Name: OLEAUT32.dll
DLL Name: SETUPAPI.dll
DLL Name: SHELL32.dll
DLL Name: SHLWAPI.dll
DLL Name: USER32.dll
DLL Name: VERSION.dll
DLL Name: AVICAP32.dll
DLL Name: WINMM.dll
DLL Name: WS2_32.dll

~/ffmpeg-build/ffmpeg-5.1.1# x86_64-w64-mingw32-objdump -p ffprobe.exe | rg 'DLL Name'
DLL Name: ADVAPI32.dll
DLL Name: bcrypt.dll
DLL Name: CRYPT32.dll
DLL Name: GDI32.dll
DLL Name: IMM32.dll
DLL Name: KERNEL32.dll
DLL Name: msvcrt.dll
DLL Name: ole32.dll
DLL Name: OLEAUT32.dll
DLL Name: SETUPAPI.dll
DLL Name: SHELL32.dll
DLL Name: SHLWAPI.dll
DLL Name: USER32.dll
DLL Name: VERSION.dll
DLL Name: AVICAP32.dll
DLL Name: WINMM.dll
DLL Name: WS2_32.dll
```


# volume の中身を別のボリュームにコピーする方法
```shell
docker run --rm -v apt_cacher_ng:/src -v apt-cacher-ng_apt-cacher-ng:/dst alpine sh -c "cd /src && cp -av . /dst"
```


# 利用ライブラリ

各ライブラリのパッチは、[MSYS2 Packages](https://packages.msys2.org) の
該当パッケージの ```Source Files``` をクリックすると、
パッチなどが表示されるので、それを参考にした。
ビルドスクリプトなども参考となる。

- [akiyo_cif.y4m](https://media.xiph.org/video/derf/)
- [AMF-1.4.26.tar.gz](https://github.com/GPUOpen-LibrariesAndSDKs/AMF)
- [aom-bcfe6fbfed315f83ee8a95465c654ee8078dbff9.tar.gz](https://aomedia.googlesource.com/aom/)
- [AviSynthPlus-3.7.2.tar.gz](https://avs-plus.net/)
- [bzip2-1.0.8.tar.gz](https://sourceware.org/bzip2/)
- [fdk-aac-2.0.2.zip](https://sourceforge.net/projects/opencore-amr/)
- [ffmpeg-5.1.1.tar.xz](http://ffmpeg.org/)
- [giflib-5.2.1.tar.gz](https://sourceforge.net/projects/giflib/)
- [glslang-11.11.0.tar.gz](https://github.com/KhronosGroup/glslang) (vulkan の依存ライブラリ)
- [gmp-6.2.1.tar.lz](https://gmplib.org/) (gnutls の依存ライブラリ)
- [gpac-1.0.1.tar.gz](https://gpac.wp.imt.fr/)
- [gsm-1.0.22.tar.gz](https://www.quut.com/gsm/)
- [gnutls-3.6.16.tar.xz](https://www.gnutls.org/)
- [lame-3.100.tar.gz](https://lame.sourceforge.io/)
- [lcms2-2.13.1.tar.gz](https://sourceforge.net/projects/lcms/)
- [libepoxy-1.5.10.tar.gz](https://github.com/anholt/libepoxy) (libplacebo の依存ライブラリ)
- [libjpeg-turbo-2.1.4.tar.gz](https://sourceforge.net/projects/libjpeg-turbo/)
- [libogg-1.3.5.tar.xz](https://www.xiph.org/ogg/)
- [libplacebo-v4.208.0.tar.bz2](https://code.videolan.org/videolan/libplacebo)
- [libpng-1.6.37.tar.xz](http://www.libpng.org/pub/png/libpng.html)
- [libtasn1-4.19.0.tar.gz](https://www.gnu.org/software/libtasn1/) (gnutls の依存ライブラリ)
- [libtheora-1.1.1.tar.bz2](https://theora.org/)
- [libunistring-1.0.tar.gz](https://www.gnu.org/software/libunistring/) (gnutls の依存ライブラリ)
- [libvorbis-1.3.7.tar.xz](https://www.xiph.org/vorbis/)
- [libvpx-1.12.0.tar.gz](https://chromium.googlesource.com/webm/libvpx)
- [libwebp-1.2.4.tar.gz](https://developers.google.com/speed/webp)
- [mfx_dispatch-1.35.1.tar.gz](https://github.com/lu-zero/mfx_dispatch)
- [nettle-3.8.tar.gz](https://www.lysator.liu.se/~nisse/nettle/) (gnutls の依存ライブラリ)
- [nv-codec-headers-n11.1.5.1.tar.gz](https://github.com/FFmpeg/nv-codec-headers)
- [OpenCL-Headers-2022.09.23.tar.gz](https://github.com/KhronosGroup/OpenCL-Headers/) (OpenCL-ICD-Loader の依存ライブラリ)
- [OpenCL-ICD-Loader-2022.09.23.tar.gz](https://github.com/KhronosGroup/OpenCL-ICD-Loader)
- [opencore-amr-0.1.6.tar.gz](https://sourceforge.net/projects/opencore-amr/)
- [openjpeg-2.5.0.tar.gz](http://www.openjpeg.org/)
- [rtmpdump-f1b83c1.tar.gz](http://rtmpdump.mplayerhq.hu/)
- [SDL2-2.24.0.tar.gz](https://www.libsdl.org/)
- [shaderc-2022.2.tar.gz](https://github.com/google/shaderc)
- [speex-1.2.0.tar.gz](https://speex.org/)
- [SPIRV-Headers-sdk-1.3.224.1.tar.gz](https://github.com/KhronosGroup/SPIRV-Headers) (vulkan の依存ライブラリ)
- [SPIRV-Tools-2022.3.tar.gz](https://github.com/KhronosGroup/SPIRV-Tools) (vulkan の依存ライブラリ)
- [tiff-4.4.0.tar.xz](http://www.libtiff.org/)
- [Vulkan-Headers-1.3.230.tar.gz](https://github.com/KhronosGroup/Vulkan-Headers)
- [Vulkan-Loader-1.3.230.tar.gz](https://github.com/KhronosGroup/Vulkan-Loader)
- [x264-stable.tar.bz2](http://x264.nl/)
- [x265_3.2.1.tar.gz](https://bitbucket.org/multicoreware/x265_git/wiki/Home)
- [xvidcore-1.3.7.tar.gz](https://www.xvid.com/)
- [xz-5.2.6.tar.xz](https://tukaani.org/xz/)
- [zlib-1.2.11.tar.xz](http://www.zlib.net/)


# ffmpeg 備忘録

## コーデックのオプション表示

```shell
ffmpeg -h encoder=hevc_amf
```

## 対応しているハードウェア支援の一覧を表示

```shell
ffmpeg -hide_banner -hwaccels

Hardware acceleration methods:
cuda
dxva2
qsv
d3d11va
opencl
vulkan
```

## 利用可能なハードウェアデバイスを表示

```shell
ffmpeg -hide_banner -v verbose -init_hw_device opencl
ffmpeg -hide_banner -v verbose -init_hw_device vulkan
ffmpeg -hide_banner -v verbose -init_hw_device cuda
ffmpeg -hide_banner -v verbose -init_hw_device qsv
```

## 入力デバイスを利用する

### 利用可能な入力デバイス一覧

```shell
ffmpeg -list_devices true -f dshow -i nul
```

### デバイスの対応フォーマットなどの一覧

```shell
ffmpeg -list_options true -f dshow -i video="HD Pro Webcam C920"
ffmpeg -list_options true -f dshow -i video="OBS Virtual Camera"
ffmpeg -list_options true -f dshow -i audio="ライン入力 (TASCAM US-366)"
```

### 入力デバイスの画像を画面に表示する

### SDL で表示する

```shell
ffmpeg -rtbufsize 5M -f dshow -c:v mjpeg -video_size 1920x1080 -framerate 30 -i video="HD Pro Webcam C920" -pix_fmt argb -f sdl "WebCam View"
ffmpeg -rtbufsize 5M -f dshow -pixel_format yuyv422 -video_size 640x480 -framerate 30 -i video="HD Pro Webcam C920" -pix_fmt argb -f sdl "WebCam View"


ffmpeg -hwaccel auto -rtbufsize 5M -f dshow -c:v h264 -video_size 1920x1080 -framerate 30 -i video="HD Pro Webcam C920" -pix_fmt argb -f sdl "WebCam View"
```

### OpenGL で表示する

```shell
ffmpeg -re -i 6289-7-20220518-0030-16.m2t -f opengl -
ffmpeg -rtbufsize 5M -f dshow -c:v mjpeg -video_size 1920x1080 -framerate 30 -i video="HD Pro Webcam C920" -pix_fmt argb -f sdl "WebCam View"
```


```shell
ffmpeg -rtbufsize 5M -f dshow -pixel_format yuv420p -video_size 1920x1080 -framerate 30 -i video="OBS Virtual Camera" -c:v copy -f sdl "SDL output"

ffmpeg -re -i 6289-7-20220518-0030-16.m2t -f sdl -
ffmpeg -rtbufsize 30M -threads 1 -init_hw_device opencl=gpu:0.0 -filter_hw_device gpu -f dshow -c:v mjpeg -video_size 1280x720 -framerate 30 -i video="HD Pro Webcam C920" -vf "format=rgba,hwupload,sobel_opencl=15:1:0,hwdownload,format=rgba" -f sdl -



ffmpeg -hwaccel dxva2 -threads 1 -i 6289-7-20220518-0030-16.m2t -f null - -benchmark
ffmpeg -threads 12 -i 6289-7-20220518-0030-16.m2t -s 1280x720 -c:v hevc_amf -quality quality -rc cqp -qmin 0 -qmax 0 -c:a copy -bsf:a aac_adtstoasc -map 0:22 -map 0:29 -y bbb.mp4

1280x720

-vf scale=1280:-1
ffmpeg -list_devices true -f dshow -i nul
ffmpeg -list_options true -f dshow -i video="HD Pro Webcam C920"

ffmpeg -hide_banner -v verbose -init_hw_device opencl
```

