# syntax=docker/dockerfile:1
FROM ubuntu:jammy

ARG THREADS=6

# disable "debconf: delaying package configuration, since apt-utils is not installed" message
ENV DEBCONF_NOWARNINGS=yes

RUN --mount=type=cache,target=/var/lib/apt/lists \
    --mount=type=cache,target=/var/cache/apt/archives \
    set -eu \
# apt リポジトリをプロキシ経由に設定
 && echo 'Acquire::HTTP::Proxy "http://gateway.docker.internal:3142";' >> /etc/apt/apt.conf.d/01proxy \
 && echo 'Acquire::HTTPS::Proxy "false";' >> /etc/apt/apt.conf.d/01proxy \
# apt インストール設定
 && echo 'APT::Install-Recommends 0;' >> /etc/apt/apt.conf.d/01norecommends \
 && echo 'APT::Install-Suggests 0;' >> /etc/apt/apt.conf.d/01norecommends \
# apt リポジトリを jaist に変更
 && sed -i.bak -s 's%http://archive.ubuntu.com/ubuntu/%http://ftp.jaist.ac.jp/pub/Linux/ubuntu/%g' /etc/apt/sources.list \
 && apt-get update \
 && apt-get -y install --no-install-recommends \
    mingw-w64 \
    mingw-w64-tools \
    make \
    cmake \
    autoconf \
    automake \
    autopoint \
    pkg-config \
    libtool \
    nasm \
    yasm \
    ccache \
    git \
    unzip \
    bzip2 \
    xz-utils \
    vim \
    ripgrep \
    # for x264
    wine \
    # for zlib
    checkinstall \
# mingw コマンドを ccache 経由で実行するリンクを生成
 && ln -s ../../bin/ccache /usr/lib/ccache/i686-w64-mingw32-cpp \
 && ln -s ../../bin/ccache /usr/lib/ccache/i686-w64-mingw32-c++ \
 && ln -s ../../bin/ccache /usr/lib/ccache/i686-w64-mingw32-gcc \
 && ln -s ../../bin/ccache /usr/lib/ccache/i686-w64-mingw32-g++ \
 && ln -s ../../bin/ccache /usr/lib/ccache/x86_64-w64-mingw32-cpp \
 && ln -s ../../bin/ccache /usr/lib/ccache/x86_64-w64-mingw32-c++ \
 && ln -s ../../bin/ccache /usr/lib/ccache/x86_64-w64-mingw32-gcc \
 && ln -s ../../bin/ccache /usr/lib/ccache/x86_64-w64-mingw32-g++

COPY libs/ /root/ffmpeg-build/
WORKDIR /root/ffmpeg-build

# 各ソース圧縮ファイルを解凍
RUN find ./ -type f \( -name "*.tar.gz" -or -name "*.tgz" \) -not \( -name "libvpx-1.12.0.tar.gz" -or -name "aom-3.5.0.tar.gz" \) | xargs -n 1 tar zxf \
 && mkdir libvpx \
 && tar zxf libvpx-1.12.0.tar.gz -C libvpx \
 && mkdir aom \
 && tar zxf aom-3.5.0.tar.gz -C aom \
 && find ./ -type f -name "*.tar.bz2" | xargs -n 1 tar jxf \
 && find ./ -type f -name "*.tar.xz" | xargs -n 1 tar Jxf \
 && find ./ -type f -name "*.zip" | xargs -n 1 unzip

RUN --mount=type=cache,target=/root/.ccache \
    set -eu \
# ccache を有効化
 && export PATH="/usr/lib/ccache:$PATH" \
# zlib
 && cd zlib-1.2.11 \
 && CFLAGS="-pipe -O3 -mtune=native -march=native" \
    CC=x86_64-w64-mingw32-gcc \
    AR=x86_64-w64-mingw32-ar \
    RANLIB=x86_64-w64-mingw32-ranlib \
    ./configure \
    --static \
    --prefix=/usr/x86_64-w64-mingw32 \
 && make -j${THREADS} \
 && checkinstall --pkgname=zlib-mingw-w64-x86-64 \
    --pkgversion="1.2.11.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# bzip2
 && cd bzip2-1.0.8 \
 && make -j${THREADS} libbz2.a \
    CC=x86_64-w64-mingw32-gcc \
    AR=x86_64-w64-mingw32-ar \
    RANLIB=x86_64-w64-mingw32-ranlib \
    PREFIX=/usr/x86_64-w64-mingw32 \
    CFLAGS="-Wall -Winline -g -D_FILE_OFFSET_BITS=64 -pipe -O3 -mtune=native -march=native" \
 && install -m 644 bzlib.h /usr/x86_64-w64-mingw32/include \
 && install -m 644 libbz2.a /usr/x86_64-w64-mingw32/lib \
 && cd .. \
# XZ Utils
 && cd xz-5.2.6 \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
 && make -j${THREADS} \
 && checkinstall --pkgname=xz-utils-mingw-w64-x86-64 \
    --pkgversion="5.2.6.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# gmp (gnutls)
 && cd gmp-6.2.1 \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
    --enable-static \
 && make -j${THREADS} \
 && checkinstall --pkgname=libgmp-mingw-w64-x86-64 \
    --pkgversion="6.2.1.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# nettle (gnutls)
 && cd nettle-3.8 \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
    --enable-static \
 && make -j${THREADS} \
 && checkinstall --pkgname=libnettle-mingw-w64-x86-64 \
    --pkgversion="3.8.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# libtasn1 (gnutls)
 && cd libtasn1-4.19.0 \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
 && make -j${THREADS} \
 && checkinstall --pkgname=libtasn1-mingw-w64-x86-64 \
    --pkgversion="4.19.0.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# libunistring (gnutls)
 && cd libunistring-1.0 \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
 && make -j${THREADS} \
 && checkinstall --pkgname=libunistring-mingw-w64-x86-64 \
    --pkgversion="1.0.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# gnutls
 && cd gnutls-3.6.16 \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    CPPFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
    --enable-static \
    --without-p11-kit \
    --disable-tests \
    --disable-doc \
 && make -j${THREADS} \
 && checkinstall --pkgname=libgnutls-mingw-w64-x86-64 \
    --pkgversion="3.6.16.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# SDL
 && cd SDL2-2.24.0 \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    CXXFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
 && make -j${THREADS} \
 && checkinstall --pkgname=libsdl-mingw-w64-x86-64 \
    --pkgversion="2.24.0.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# gpac
 && cd gpac-1.0.1 \
 && patch -p0 < ../gpac-1.0.1.patch \
 && ./configure \
    --prefix=/usr/x86_64-w64-mingw32 \
    --cross-prefix=x86_64-w64-mingw32- \
    --cpu=x86_64 \
    --target-os=mingw64 \
    --extra-cflags='-pipe -O3 -mtune=native -march=native -mfpmath=both' \
    --extra-ldflags='-static-libstdc++ -static-libgcc' \
    --sdl-cfg=/usr/x86_64-w64-mingw32/bin \
    --enable-sdl-static \
    --static-mp4box \
    --enable-static-bin \
 && make -j${THREADS} \
 && checkinstall --pkgname=libgpac-mingw-w64-x86-64 \
    --pkgversion="1.0.1.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default make install-lib \
 && cd .. \
# AviSynthPlus
 && cd AviSynthPlus-3.7.2 \
 && mkdir avisynth-build \
 && cd avisynth-build \
 && cmake ../ \
    -DHEADERS_ONLY:bool=on \
    -DCMAKE_INSTALL_PREFIX=/usr/x86_64-w64-mingw32 \
    -DCMAKE_TOOLCHAIN_FILE=../../TC-mingw.cmake \
    -DCMAKE_C_FLAGS="-pipe -O3 -mtune=native -march=native" \
    -DCMAKE_CXX_FLAGS="-pipe -O3 -mtune=native -march=native" \
 && make -j${THREADS} VersionGen \
 && checkinstall --pkgname=libavisynth-mingw-w64-x86-64 \
    --pkgversion="3.7.2.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd ../.. \
# aom
 && cd aom \
 && mkdir build-work \
 && cd build-work \
 && cmake ../ \
    -DCMAKE_INSTALL_PREFIX=/usr/x86_64-w64-mingw32 \
    -DCMAKE_TOOLCHAIN_FILE=../../TC-mingw.cmake \
    -DCMAKE_C_FLAGS="-pipe -O3 -mtune=native -march=native" \
    -DCMAKE_CXX_FLAGS="-pipe -O3 -mtune=native -march=native" \
 && make -j${THREADS} \
 && checkinstall --pkgname=libaom-mingw-w64-x86-64 \
    --pkgversion="3.5.0.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd ../.. \
# x264
 && cd x264-stable \
 && ./configure \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --cross-prefix=x86_64-w64-mingw32- \
    --enable-static \
    --extra-cflags="-pipe -O3 -mtune=native -march=native -mfpmath=both" \
 && patch < ../x264-stable.patch \
 && make -j${THREADS} fprofiled VIDS=../akiyo_cif.y4m \
 && checkinstall --pkgname=libx264-mingw-w64-x86-64 \
    --pkgversion="2334.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd ..\
# x265
 && cd x265_3.2.1 \
 && mkdir -p build/mingw \
 && cd build/mingw \
 && cmake ../../source \
    -DCMAKE_INSTALL_PREFIX=/usr/x86_64-w64-mingw32 \
    -DCMAKE_TOOLCHAIN_FILE=../../../TC-mingw.cmake \
    -DCMAKE_C_FLAGS="-pipe -O3 -mtune=native -march=native" \
    -DCMAKE_CXX_FLAGS="-pipe -O3 -mtune=native -march=native" \
    -DENABLE_SHARED=OFF \
 && make -j${THREADS} \
 && checkinstall --pkgname=libx265-mingw-w64-x86-64 \
    --pkgversion="3.2.1.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd ../../.. \
# libpng
 && cd libpng-1.6.37 \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
 && make -j${THREADS} \
 && checkinstall --pkgname=libpng-mingw-w64-x86-64 \
    --pkgversion="1.6.37.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# tiff
 && cd tiff-4.4.0 \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    CPPFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
 && make -j${THREADS} \
 && checkinstall --pkgname=libtiff-mingw-w64-x86-64 \
    --pkgversion="4.4.0.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# libjpeg-turbo
 && cd libjpeg-turbo-2.1.4 \
 && mkdir libjpeg-turbo-build \
 && cd libjpeg-turbo-build \
 && cmake ../ \
    -G"Unix Makefiles" \
    -DCMAKE_INSTALL_PREFIX=/usr/x86_64-w64-mingw32 \
    -DCMAKE_TOOLCHAIN_FILE=../../TC-mingw.cmake \
    -DCMAKE_C_FLAGS="-pipe -O3 -mtune=native -march=native" \
    -DENABLE_SHARED=FALSE \
    -DENABLE_STATIC=TRUE \
 && make -j${THREADS} \
 && checkinstall --pkgname=libjpeg-turbo-mingw-w64-x86-64 \
    --pkgversion="2.1.4.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd ../.. \
# giflib \
 && cd giflib-5.2.1 \
 && patch < ../giflib-5.2.1.patch \
 && make \
    -j${THREADS} \
    libgif.a \
    OFLAGS="-pipe -O3 -mtune=native -march=native" \
    CC=x86_64-w64-mingw32-gcc \
    AR=x86_64-w64-mingw32-ar \
 && checkinstall --pkgname=libgif-mingw-w64-x86-64 \
    --pkgversion="5.2.1.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
    make install-include install-lib \
    PREFIX=/usr/x86_64-w64-mingw32 \
 && cd .. \
# libwebp
 && cd libwebp-1.2.4 \
 && ./configure \
    LIBS="-lpng -lz" \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
 && make -j${THREADS} \
 && checkinstall --pkgname=libwebp-mingw-w64-x86-64 \
    --pkgversion="1.2.4.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# lcms2
 && cd lcms2-2.13.1 \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    CPPFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
 && make -j${THREADS} \
 && checkinstall --pkgname=liblcms2-mingw-w64-x86-64 \
    --pkgversion="2.13.1.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# lame
 && cd lame-3.100 \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    CPPFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
    --enable-nasm \
 && make -j${THREADS} \
 && checkinstall --pkgname=lame-mingw-w64-x86-64 \
    --pkgversion="3.100.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# libogg \
 && cd libogg-1.3.5 \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
 && make -j${THREADS} \
 && checkinstall --pkgname=libogg-mingw-w64-x86-64 \
    --pkgversion="1.3.5.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# libvorbis
 && cd libvorbis-1.3.7 \
 && ./autogen.sh \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
 && make -j${THREADS} \
 && checkinstall --pkgname=libvorbis-mingw-w64-x86-64 \
    --pkgversion="1.3.7.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# libtheora
 && cd libtheora-1.1.1 \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
    --disable-examples \
 && make -j${THREADS} \
 && checkinstall --pkgname=libtheora-mingw-w64-x86-64 \
    --pkgversion="1.1.1.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# speex
 && cd speex-1.2.0 \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
    --enable-sse \
 && make -j${THREADS} \
 && checkinstall --pkgname=libspeex-mingw-w64-x86-64 \
    --pkgversion="1.2.0.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# opencore-amr
 && cd opencore-amr-0.1.6 \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    CXXFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
 && make -j${THREADS} \
 && checkinstall --pkgname=libopencore-amr-mingw-w64-x86-64 \
    --pkgversion="0.1.6.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# openjpeg
 && cd openjpeg-2.5.0 \
 && mkdir build \
 && cd build \
 && cmake ../ \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr/x86_64-w64-mingw32 \
    -DCMAKE_TOOLCHAIN_FILE=../../TC-mingw.cmake \
    -DCMAKE_C_FLAGS="-pipe -O3 -mtune=native -march=native" \
    -DBUILD_SHARED_LIBS=OFF \
    -DBUILD_STATIC_LIBS=ON \
 && make -j${THREADS} \
 && checkinstall --pkgname=libopenjpeg-mingw-w64-x86-64 \
    --pkgversion="2.5.0.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd ../.. \
# xvidcore
 && cd xvidcore/build/generic \
 && patch < ../../../xvidcore-1.3.7.patch \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    CPPFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-pthread \
 && make -j${THREADS} \
 && checkinstall --pkgname=libxvidcore-mingw-w64-x86-64 \
    --pkgversion="1.3.7.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd ../../.. \
# gsm
 && cd gsm-1.0-pl22 \
 && mkdir -p /usr/x86_64-w64-mingw32/man/man3 \
 && checkinstall --pkgname=libgsm-mingw-w64-x86-64 \
    --pkgversion="1.0.22.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
    --exclude=/root \
    make \
    -j${THREADS} \
    gsminstall \
    INSTALL_ROOT="/usr/x86_64-w64-mingw32" \
    CC="x86_64-w64-mingw32-gcc -ansi -pedantic" \
    CCFLAGS="-c -DNeedFunctionPrototypes=1 -Wall -Wno-comment -pipe -O3 -mtune=native -march=native" \
    AR=x86_64-w64-mingw32-ar \
    RANLIB=x86_64-w64-mingw32-ranlib \
    GSM_INSTALL_INC="/usr/x86_64-w64-mingw32/include" \
 && cd .. \
# libvpx
 && cd libvpx \
 && CROSS=x86_64-w64-mingw32- \
    ./configure \
    --extra-cflags="-pipe -O3 -mtune=native -march=native" \
    --extra-cxxflags="-pipe -O3 -mtune=native -march=native" \
    --target=x86_64-win64-gcc \
    --prefix=/usr/x86_64-w64-mingw32 \
    --disable-shared \
    --disable-examples \
 && make -j${THREADS} \
 && checkinstall --pkgname=libvpx-mingw-w64-x86-64 \
    --pkgversion="1.12.0.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# fdk-aac
 && cd fdk-aac-2.0.2 \
 && ./autogen.sh \
 && ./configure \
    CFLAGS="-pipe -O3 -mtune=native -march=native" \
    CPPFLAGS="-pipe -O3 -mtune=native -march=native" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --host=x86_64-w64-mingw32 \
    --disable-shared \
    --enable-static \
 && make -j${THREADS} \
 && checkinstall --pkgname=libfdk-aac-mingw-w64-x86-64 \
    --pkgversion="2.0.2.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd .. \
# ffmpeg
 && cd ffmpeg-5.1.1 \
 && ./configure \
    --ld=x86_64-w64-mingw32-g++ \
    --extra-libs="-lvorbis -logg -ltasn1 -lhogweed -lnettle -lgmp -lunistring -lcrypt32 -lws2_32 -lpthread" \
    --extra-cflags="-pipe -O3 -mtune=native -march=native -mfpmath=both" \
    --extra-cxxflags="-pipe -O3 -mtune=native -march=native -mfpmath=both" \
    --extra-ldflags="-static" \
    --prefix=/usr/x86_64-w64-mingw32 \
    --cross-prefix=x86_64-w64-mingw32- \
    --target-os=mingw32 \
    --arch=x86_64 \
    --cpu=native \
    --enable-gpl \
    --enable-version3 \
    --enable-nonfree \
    --enable-postproc \
    --enable-runtime-cpudetect \
    --enable-avisynth \
    --enable-gmp \
    --enable-gnutls \
    --enable-lcms2 \
    --enable-libaom \
    --enable-libfdk-aac \
    --enable-libgsm \
    --enable-libmp3lame \
    --enable-libopencore-amrnb \
    --enable-libopencore-amrwb \
    --enable-libopenjpeg \
    --enable-libspeex \
    --enable-libtheora \
    --enable-libvorbis \
    --enable-libwebp \
    --enable-libx264 \
    --enable-libx265 \
    --enable-libxvid \
    --enable-zlib \
    --enable-bzlib \
    --disable-debug \
    --disable-pthreads \
    --enable-static \
    --disable-shared \
 && make -j${THREADS} \
 && checkinstall --pkgname=libffmpeg-mingw-w64-x86-64 \
    --pkgversion="5.1.1.$(date +%Y%m%d)-0ubuntu1" \
    --backup=no --deldoc=yes --fstrans=no --default \
 && cd ..

VOLUME ["/root/.ccache"]
