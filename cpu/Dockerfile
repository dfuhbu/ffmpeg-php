FROM php:8.0.8-cli-buster

ENV TZ=UTC

ENV NV_CODEC_HEADERS=n11.0.10.1
# x264 http://www.videolan.org/developers/x264.html
ENV X264_VERSION=stable
# x265 http://x265.org/
ENV X265_VERSION=3.2.1
# libogg https://www.xiph.org/ogg/
ENV OGG_VERSION=1.3.5
# libopus https://www.opus-codec.org/
ENV OPUS_VERSION=1.3.1
# libtheora http://www.theora.org/
ENV THEORA_VERSION=1.1.1
# libvorbis https://xiph.org/vorbis/
ENV VORBIS_VERSION=1.3.7
# libvpx https://www.webmproject.org/code/
ENV VPX_VERSION=1.10.0
# libwebp https://storage.googleapis.com/downloads.webmproject.org/releases/webp/index.html
ENV WEBP_VERSION=1.2.0
# libmp3lame http://lame.sourceforge.net/
# libmp3lame https://sourceforge.net/projects/lame/files/lame/
ENV LAME_VERSION=3.100
# fdk-aac https://github.com/mstorsjo/fdk-aac
ENV FDKAAC_VERSION=v2.0.2
# kvazaar https://github.com/ultravideo/kvazaar
ENV KVAZAAR_VERSION=v2.0.0
# https://xcb.freedesktop.org/dist/
ENV LIBPTHREAD_STUBS_VERSION=0.4
# ffmpeg https://ffmpeg.org/
# ffmpeg https://ffmpeg.org/releases/
ENV FFMPEG_VERSION=4.4
# https://github.com/Netflix/vmaf
ENV VMAF=v2.2.0

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
    && echo $TZ > /etc/timezone \
    && DEV_IM_APP="libpng16-16 libpng-dev libjpeg62-turbo libjpeg62-turbo-dev libgomp1 ghostscript libxml2-dev libxml2-utils" \
    && DEV_APP="autoconf automake cmake curl bzip2 ninja-build libexpat1-dev g++ gcc git gperf libtool make nasm perl pkg-config python3 python3-pip python3-setuptools python3-wheel libssl-dev yasm zlib1g-dev" \
    && DEBIAN_FRONTEND=noninteractive \
    && apt-get -yqq update \
    && apt-get install -yq --no-install-recommends apt-transport-https ca-certificates $DEV_APP $DEV_IM_APP \
    && pip3 install --user meson numpy cython \
    # settings
    && export PREFIX=/opt/ffmpeg \
    && export LD_LIBRARY_PATH=/opt/ffmpeg/lib \
    && export MAKEFLAGS="-j2" \
    && export PKG_CONFIG_PATH="/opt/ffmpeg/share/pkgconfig:/opt/ffmpeg/lib/pkgconfig:/opt/ffmpeg/lib64/pkgconfig" \
    && export LD_LIBRARY_PATH="/opt/ffmpeg/lib:/opt/ffmpeg/lib64:/usr/lib64:/usr/lib:/lib64:/lib" \
    && export PATH="$PATH:$PREFIX/bin" \
    # nv codec headers
    && cd /tmp/ \
    && NV_CODEC_HEADERS_DIR=/tmp/nv-codec-headers/ \
    && rm -rf $NV_CODEC_HEADERS_DIR \
    && mkdir -p $NV_CODEC_HEADERS_DIR \
    && cd $NV_CODEC_HEADERS_DIR \
    && git clone --depth 1 --branch $NV_CODEC_HEADERS https://github.com/FFmpeg/nv-codec-headers $NV_CODEC_HEADERS_DIR \
    && make PREFIX="$PREFIX" \
    && make install PREFIX="$PREFIX" \
    && rm -rf $NV_CODEC_HEADERS_DIR \
    # H264
    && cd /tmp/ \
    && H254_DIR=/tmp/x264/ \
    && rm -rf $H254_DIR \
    && mkdir -p $H254_DIR \
    && cd $H254_DIR \
    && git clone --depth 1 --branch $X264_VERSION https://code.videolan.org/videolan/x264.git $H254_DIR \
    && ./configure --prefix="$PREFIX" --enable-shared --enable-pic --disable-cli \
    && make \
    && make install \
    && rm -rf $H254_DIR \
    # H265
    && cd /tmp/ \
    && H265_DIR=/tmp/x265 \
    && rm -rf $H265_DIR \
    && mkdir -p $H265_DIR \
    && cd $H265_DIR \
    && curl -sL https://download.videolan.org/pub/videolan/x265/x265_${X265_VERSION}.tar.gz | tar -zx \
    && cd x265_${X265_VERSION}/build/linux \
    && sed -i "/-DEXTRA_LIB/ s/$/ -DCMAKE_INSTALL_PREFIX=\${PREFIX}/" multilib.sh \
    && sed -i "/^cmake/ s/$/ -DENABLE_CLI=OFF/" multilib.sh \
    && chmod u+x ./multilib.sh \
    && ./multilib.sh \
    && make -C 8bit install \
    && rm -rf $H265_DIR \
    # Ogg
    && cd /tmp/ \
    && OGG_DIR=/tmp/ogg \
    && rm -rf $OGG_DIR \
    && mkdir -p $OGG_DIR \
    && cd $OGG_DIR \
    && curl -sLO http://downloads.xiph.org/releases/ogg/libogg-${OGG_VERSION}.tar.gz \
    && tar -zx --strip-components=1 -f libogg-${OGG_VERSION}.tar.gz \
    && ./configure --prefix="${PREFIX}" --enable-shared \
    && make \
    && make install \
    && rm -rf $OGG_DIR \
    # Opus
    && cd /tmp/ \
    && OPUS_DIR=/tmp/opus \
    && rm -rf $OPUS_DIR \
    && mkdir -p $OPUS_DIR \
    && cd $OPUS_DIR \
    && curl -sLO https://archive.mozilla.org/pub/opus/opus-${OPUS_VERSION}.tar.gz \
    && tar -zx --strip-components=1 -f opus-${OPUS_VERSION}.tar.gz \
    && autoreconf -fiv \
    && ./configure --prefix="${PREFIX}" --enable-shared \
    && make \
    && make install \
    && rm -rf $OPUS_DIR \
    # Theora
    && cd /tmp/ \
    && THEORA_DIR=/tmp/theora \
    && rm -rf $THEORA_DIR \
    && mkdir -p $THEORA_DIR \
    && cd $THEORA_DIR \
    && curl -sLO http://downloads.xiph.org/releases/theora/libtheora-${THEORA_VERSION}.tar.gz \
    && tar -zx --strip-components=1 -f libtheora-${THEORA_VERSION}.tar.gz \
    && ./configure --prefix="${PREFIX}" --with-ogg="${PREFIX}" --enable-shared \
    && make \
    && make install \
    && rm -rf $THEORA_DIR \
    # Vorbis
    && cd /tmp/ \
    && VORBIS_DIR=/tmp/vorbis \
    && rm -rf $VORBIS_DIR \
    && mkdir -p $VORBIS_DIR \
    && cd $VORBIS_DIR \
    && curl -sLO http://downloads.xiph.org/releases/vorbis/libvorbis-${VORBIS_VERSION}.tar.gz \
    && tar -zx --strip-components=1 -f libvorbis-${VORBIS_VERSION}.tar.gz \
    && ./configure --prefix="${PREFIX}" --with-ogg="${PREFIX}" --enable-shared \
    && make \
    && make install \
    && rm -rf $VORBIS_DIR \
    # Vpx
    && cd /tmp/ \
    && VPX_DIR=/tmp/vpx \
    && rm -rf $VPX_DIR \
    && mkdir -p $VPX_DIR \
    && cd $VPX_DIR \
    && curl -sL https://codeload.github.com/webmproject/libvpx/tar.gz/v${VPX_VERSION} | tar -zx --strip-components=1 \
    && ./configure --prefix="${PREFIX}" --enable-vp8 --enable-vp9 --enable-vp9-highbitdepth --enable-pic --enable-shared --disable-debug --disable-examples --disable-docs --disable-install-bins \
    && make \
    && make install \
    && rm -rf $VPX_DIR \
    # Webp
    && cd /tmp/ \
    && WEBP_DIR=/tmp/webp \
    && rm -rf $WEBP_DIR \
    && mkdir -p $WEBP_DIR \
    && cd $WEBP_DIR \
    && curl -sL https://storage.googleapis.com/downloads.webmproject.org/releases/webp/libwebp-${WEBP_VERSION}.tar.gz | tar -zx --strip-components=1 \
    && ./configure --prefix="${PREFIX}" --enable-shared \
    && make \
    && make install \
    && rm -rf $WEBP_DIR \
    # Mp3
    && cd /tmp/ \
    && MP3_DIR=/tmp/lame \
    && rm -rf $MP3_DIR \
    && mkdir -p $MP3_DIR \
    && cd $MP3_DIR \
    && curl -sL https://versaweb.dl.sourceforge.net/project/lame/lame/$(echo ${LAME_VERSION} | sed -e 's/[^0-9]*\([0-9]*\)[.]\([0-9]*\)[.]\([0-9]*\)\([0-9A-Za-z-]*\)/\1.\2/')/lame-${LAME_VERSION}.tar.gz | tar -zx --strip-components=1 \
    && ./configure --prefix="${PREFIX}" --bindir="${PREFIX}/bin" --enable-shared --enable-nasm --disable-frontend \
    && make \
    && make install \
    && rm -rf $MP3_DIR \
    # Aac
    && cd /tmp/ \
    && ACC_DIR=/tmp/fdk-aac \
    && rm -rf $ACC_DIR \
    && mkdir -p $ACC_DIR \
    && cd $ACC_DIR \
    && git clone --depth 1 --branch $FDKAAC_VERSION https://github.com/mstorsjo/fdk-aac $ACC_DIR \
    && autoreconf -fiv \
    && ./configure --prefix="${PREFIX}" --enable-shared --datadir="$ACC_DIR" \
    && make \
    && make install \
    && rm -rf $ACC_DIR \
    # Kvazaar
    && cd /tmp/ \
    && KVAZAAR_DIR=/tmp/kvazaar \
    && rm -rf $KVAZAAR_DIR \
    && mkdir -p $KVAZAAR_DIR \
    && cd $KVAZAAR_DIR \
    && git clone --depth 1 --branch $KVAZAAR_VERSION https://github.com/ultravideo/kvazaar $KVAZAAR_DIR \
    && ./autogen.sh \
    && ./configure --prefix="${PREFIX}" --disable-static --enable-shared \
    && make \
    && make install \
    && rm -rf $KVAZAAR_DIR \
    #  Pthread stubs
    && cd /tmp/ \
    && PTHREAD_DIR=/tmp/libpthread-stubs \
    && rm -rf $PTHREAD_DIR \
    && mkdir -p $PTHREAD_DIR \
    && cd $PTHREAD_DIR \
    && curl -sLO https://xcb.freedesktop.org/dist/libpthread-stubs-${LIBPTHREAD_STUBS_VERSION}.tar.gz \
    && tar -zx --strip-components=1 -f libpthread-stubs-${LIBPTHREAD_STUBS_VERSION}.tar.gz \
    && ./configure --prefix="${PREFIX}" \
    && make \
    && make install \
    && rm -rf $PTHREAD_DIR \
    # VMAF
    && PATH="$PATH:$HOME/.local/bin" \
    && cd /tmp/ \
    && VMAF_DIR=/tmp/vmaf \
    && rm -rf $VMAF_DIR \
    && mkdir -p $VMAF_DIR \
    && git clone --depth 1 --branch $VMAF https://github.com/Netflix/vmaf $VMAF_DIR \
    && cd $VMAF_DIR \
    && meson setup libvmaf libvmaf/build --buildtype release --prefix="$PREFIX" --libdir "$PREFIX/lib" -Denable_float=true \
    && ninja -vC libvmaf/build include/vcs_version.h \
    && ninja -vC libvmaf/build install \
    && cp -r model "$PREFIX/share" \
    && rm -rf $VMAF_DIR \
    # Ffmpeg
    && cd /tmp/ \
    && FFMPEG_DIR=/tmp/ffmpeg \
    && rm -rf $FFMPEG_DIR \
    && mkdir -p $FFMPEG_DIR \
    && cd $FFMPEG_DIR \
    && curl -sLO https://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.bz2 \
    && tar -jx --strip-components=1 -f ffmpeg-${FFMPEG_VERSION}.tar.bz2 \
    && ./configure \
    --disable-debug \
    --disable-doc \
    --disable-ffplay \
    --enable-shared \
    --enable-gpl \
    --enable-nonfree \
    --enable-openssl \
    --enable-postproc \
    --enable-small \
    --enable-version3 \
    --enable-libmp3lame \
    --enable-libopus \
    --enable-libtheora \
    --enable-libvorbis \
    --enable-libvpx \
    --enable-libwebp \
    --enable-libx265 \
    --enable-libx264 \
    --enable-libkvazaar \
    --enable-libfdk_aac \
    --enable-libvmaf \
    --prefix="${PREFIX}" \
    --extra-libs=-ldl \
    --extra-libs=-lpthread \
    --extra-cflags="-I${PREFIX}/include -I${PREFIX}/include/ffnvcodec -I/usr/local/cuda/include/" \
    --extra-ldflags="-L${PREFIX}/lib -L/usr/local/cuda/lib64 -L/usr/local/cuda/lib32/" \
    && make -j4 \
    && make install \
    && make distclean \
    && hash -r \
    && rm -rf $FFMPEG_DIR \
    # clear trash
    && cd /tmp/ \
    && pip3 uninstall -y meson numpy \
    && rm -rf /usr/local/lib/python*/ \
    && rm -rf /root/.cache/ /root/.local/ \
    && apt-get remove -y $DEV_APP \
    && apt-get -y autoremove \
    && rm -rf /var/lib/apt/lists/*
