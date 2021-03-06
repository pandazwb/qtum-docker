FROM ubuntu
LABEL maintainers "earlz, VoR0220"

WORKDIR /root
RUN set -emx \
    && apt-get update \
    && apt-get install -y -qq --no-install-recommends ca-certificates curl wget apt-utils build-essential git libtool autotools-dev automake pkg-config libssl-dev libevent-dev bsdmainutils git cmake libboost-all-dev software-properties-common texinfo bison flex libmpc-dev libmpfr-dev libgmp3-dev yasm golang-go\
    && add-apt-repository ppa:bitcoin/bitcoin -y \
    && apt-get update \
    && apt-get install libdb4.8-dev libdb4.8++-dev -y -qq --no-install-recommends


ENV REPO_PREFIX=https://github.com/qtumproject
ENV BRANCH=proto-x86
ENV MAKE_SUFFIX=-j2


# get x86-toolchain source
RUN set -ex \
    && git clone $REPO_PREFIX/x86-toolchain \
    && cd x86-toolchain \
    && git checkout $BRANCH




ENV PREFIX=/opt/cross
ENV TARGET=i686-elf
ENV SYSROOT=/opt/x86-compiler/sysroot
ENV PATH=${PATH}:${SYSROOT}/bin:$PREFIX/bin:/root/qtum:/root/qtum:/root/qtum-docker/proto-x86/utils:/root/go/bin

# build the freestanding compiler
RUN set -ex \
    && cd x86-toolchain \
    && mkdir build-binutils \
    && cd build-binutils \
    && ../binutils-2.29/configure --target="$TARGET" --prefix="$PREFIX" --disable-werror \
    && make \
    && make install \
    && cd .. \
    && mkdir build-gcc \
    && cd build-gcc \
    && ../gcc-7.2.0/configure --target="$TARGET" --prefix="$PREFIX" --enable-languages=c,c++ \
    && make all-gcc \
    && make all-target-libgcc \
    && make install-gcc \
    && make install-target-libgcc

#get libc
RUN set -ex \
    && git clone $REPO_PREFIX/FsLibc \
    && cd FsLibc \
    && git checkout $BRANCH

ENV TARGET=i686-qtum

#compile QtumOS compiler and supports
RUN set -ex \
    && ls $PREFIX/bin \
    && i686-elf-gcc -v \
    && cd FsLibc \
    && cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE=cross-toolchain.cmake -DCMAKE_INSTALL_PREFIX=$SYSROOT/usr . \
    && make -C libc \
    && make -C libc install \
    && cd ~/x86-toolchain/crtfiles \
    && make \
    && cd .. \
    && mkdir -p $SYSROOT/usr/lib \
    && mkdir -p $SYSROOT/usr/include \
    && cp crtfiles/*.o $SYSROOT/usr/lib/ \
    && cp -r includes/* $SYSROOT/usr/include/ \
    && rm -rf build-binutils \
    && mkdir build-binutils \
    && cd build-binutils \
    && ../binutils-2.29/configure --target=$TARGET --prefix="$PREFIX" --with-sysroot="$SYSROOT" --disable-werror \
    && make \
    && make install \
    && cd .. \
    && rm -rf build-gcc \
    && mkdir build-gcc \
    && cd build-gcc \
    && ../gcc-7.2.0/configure --target=$TARGET --prefix="$PREFIX" --with-sysroot="$SYSROOT" --enable-languages=c \
    && make all-gcc \
    && make all-target-libgcc \
    && make install-gcc \
    && make install-target-libgcc



# get qtum source
RUN set -ex \
    && git clone $REPO_PREFIX/qtum qtum-compile --recursive \
    && cd qtum-compile \
    && git checkout $BRANCH

# build qtum
RUN set -ex \
    && cd qtum-compile \
    && ./autogen.sh \
    && ./configure --disable-tests --without-gui \
    && make $MAKE_SUFFIX

# build x86lib utility
RUN set -ex \
    && cd qtum-compile/src/x86lib \ 
    && make $MAKE_SUFFIX

#install qtum tools
RUN set -ex \
    && cd qtum-compile \
    && mkdir /root/qtum \
    && cp src/qtumd src/qtum-cli src/qtum-tx src/x86lib/x86testbench /root/qtum

# get libqtum
RUN set -ex \
    && git clone $REPO_PREFIX/libqtum \
    && cd libqtum \
    && git checkout $BRANCH

# build libqtum
RUN set -ex \
    && cd libqtum \
    && make \
    && make deploy

# get simpleabi (this url should change in the future)
RUN go get -u github.com/qtumproject/SimpleABI && go install github.com/qtumproject/SimpleABI

# get docker helpers
RUN set -ex \
    && git clone $REPO_PREFIX/qtum-docker \
    && cd qtum-docker \
    && git checkout $BRANCH

#clean up
RUN set -ex \
    && rm -rf qtum-compile \
    && rm -rf x86-toolchain

# create data directory
ENV QTUM_DATA /data
RUN mkdir $QTUM_DATA \
    && ln -sfn $QTUM_DATA /root/.qtum \
VOLUME /data

EXPOSE 3888 3889 13888 13889
CMD ["qtumd", "-regtest", "-logevents"]
