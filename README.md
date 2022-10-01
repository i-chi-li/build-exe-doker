# はじめに
このプロジェクトは、ffmpeg をビルドする。
ffmpeg の各実行ファイルは、DLL なしで単体起動できる。
Docker コンテナをビルドすると、ffmpeg の各実行ファイルが生成される。

# apt キャッシュ用コンテナ

ffmpeg をビルドする前に、apt キャッシュ用のコンテナを起動する。

```shell
cd apt-cacher-ng
docker compose up -d --build
cd ..
```

# ffmpeg ビルド用コンテナ

```shell
cd ffmpeg
docker compose up -d --build
```

完了までには数十分かかる。

完了すると、```/root/ffmpeg-build/ffmpeg-5.1.1``` に
ffmpeg.exe, ffplay.exe, ffprobe.exe などが生成される。

実行中に以下のようなメッセージが表示されるが問題ない。
```[output clipped, log limit 1MiB reached]```

# 依存 DLL チェック

```
root@be35494fd667:~/ffmpeg-build/ffmpeg-5.1.1# x86_64-w64-mingw32-objdump -p ffmpeg.exe | rg 'DLL Name'
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
root@be35494fd667:~/ffmpeg-build/ffmpeg-5.1.1# x86_64-w64-mingw32-objdump -p ffplay.exe | rg 'DLL Name'
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
root@be35494fd667:~/ffmpeg-build/ffmpeg-5.1.1# x86_64-w64-mingw32-objdump -p ffprobe.exe | rg 'DLL Name'
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

- [akiyo_cif.y4m](https://media.xiph.org/video/derf/)
- [AMF-1.4.26.tar.gz](https://github.com/GPUOpen-LibrariesAndSDKs/AMF)
- [aom-bcfe6fbfed315f83ee8a95465c654ee8078dbff9.tar.gz](https://aomedia.googlesource.com/aom/)
- [AviSynthPlus-3.7.2.tar.gz](https://avs-plus.net/)
- [bzip2-1.0.8.tar.gz](https://sourceware.org/bzip2/)
- [fdk-aac-2.0.2.zip](https://sourceforge.net/projects/opencore-amr/)
- [ffmpeg-5.1.1.tar.xz](http://ffmpeg.org/)
- [giflib-5.2.1.tar.gz](https://sourceforge.net/projects/giflib/)
- [gmp-6.2.1.tar.lz](https://gmplib.org/) (gnutls の依存ライブラリ)
- [gpac-1.0.1.tar.gz](https://gpac.wp.imt.fr/)
- [gsm-1.0.22.tar.gz](https://www.quut.com/gsm/)
- [gnutls-3.6.16.tar.xz](https://www.gnutls.org/)
- [lame-3.100.tar.gz](https://lame.sourceforge.io/)
- [lcms2-2.13.1.tar.gz](https://sourceforge.net/projects/lcms/)
- [libjpeg-turbo-2.1.4.tar.gz](https://sourceforge.net/projects/libjpeg-turbo/)
- [libogg-1.3.5.tar.xz](https://www.xiph.org/ogg/)
- [libpng-1.6.37.tar.xz](http://www.libpng.org/pub/png/libpng.html)
- [libtasn1-4.19.0.tar.gz](https://www.gnu.org/software/libtasn1/) (gnutls の依存ライブラリ)
- [libtheora-1.1.1.tar.bz2](https://theora.org/)
- [libunistring-1.0.tar.gz](https://www.gnu.org/software/libunistring/) (gnutls の依存ライブラリ)
- [libvorbis-1.3.7.tar.xz](https://www.xiph.org/vorbis/)
- [libvpx-1.12.0.tar.gz](https://chromium.googlesource.com/webm/libvpx)
- [libwebp-1.2.4.tar.gz](https://developers.google.com/speed/webp)
- [nettle-3.8.tar.gz](https://www.lysator.liu.se/~nisse/nettle/) (gnutls の依存ライブラリ)
- [OpenCL-Headers-2022.09.23.tar.gz](https://github.com/KhronosGroup/OpenCL-Headers/) (OpenCL-ICD-Loader の依存ライブラリ)
- [OpenCL-ICD-Loader-2022.09.23.tar.gz](https://github.com/KhronosGroup/OpenCL-ICD-Loader)
- [opencore-amr-0.1.6.tar.gz](https://sourceforge.net/projects/opencore-amr/)
- [openjpeg-2.5.0.tar.gz](http://www.openjpeg.org/)
- [rtmpdump-f1b83c1.tar.gz](http://rtmpdump.mplayerhq.hu/)
- [SDL2-2.24.0.tar.gz](https://www.libsdl.org/)
- [speex-1.2.0.tar.gz](https://speex.org/)
- [tiff-4.4.0.tar.xz](http://www.libtiff.org/)
- [x264-stable.tar.bz2](http://x264.nl/)
- [x265_3.2.1.tar.gz](https://bitbucket.org/multicoreware/x265_git/wiki/Home)
- [xvidcore-1.3.7.tar.gz](https://www.xvid.com/)
- [xz-5.2.6.tar.xz](https://tukaani.org/xz/)
- [zlib-1.2.11.tar.xz](http://www.zlib.net/)

