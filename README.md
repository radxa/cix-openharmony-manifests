# Cix Openharmony for Radxa Orion Series — Build & Flash Guide

This guide provides step-by-step instructions for setting up a build environment, downloading the source code, building, and flashing openharmony on the Radxa O6.

## Build environment setup

Recommend Ubuntu 22.04 64-bit as the build host

## System Requirements

Disk Space: At least 400 GB of free space for source code checkout and build.

RAM: At least 32 GB of RAM is strongly recommended for a smooth build experience.

```bash
$ mkdir -p ~/bin
$ wget 'https://storage.googleapis.com/git-repo-downloads/repo' -P ~/bin
$ chmod +x ~/bin/repo
```
## Docker environment

Recommend using Docker,The current Docker environment configuration is as follows

```bash
FROM ubuntu:22.04

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN echo "deb http://archive.ubuntu.com/ubuntu jammy main universe multiverse restricted" > /etc/apt/sources.list && \
    echo "deb http://archive.ubuntu.com/ubuntu jammy-updates main universe multiverse restricted" >> /etc/apt/sources.list && \
    echo "deb http://archive.ubuntu.com/ubuntu jammy-backports main universe multiverse restricted" >> /etc/apt/sources.list && \
    echo "deb http://security.ubuntu.com/ubuntu jammy-security main universe multiverse restricted" >> /etc/apt/sources.list

RUN apt-get update --fix-missing && \
    apt-get upgrade -y && \
    apt-get install -y --fix-broken \
        binutils git git-lfs gnupg flex bison gperf \
        build-essential zip curl zlib1g-dev pkg-config lz4 cpio \
    && rm -rf /var/lib/apt/lists/*

RUN dpkg --add-architecture i386 && \
    apt-get update --fix-missing && \
    apt-get install -y --fix-broken \
        gcc-multilib g++-multilib libc6-dev-i386 \
        libncurses5-dev:i386 x11proto-core-dev libx11-dev \
        lib32z1-dev libssl-dev liblz4-dev \
    && rm -rf /var/lib/apt/lists/*

RUN apt-get update --fix-missing && \
    apt-get install -y --fix-broken \
        ruby ccache libgl1-mesa-dev libxml2-utils xsltproc \
        unzip m4 bc gnutls-bin python3-pip wget \
    && rm -rf /var/lib/apt/lists/*

RUN apt-get update --fix-missing && \
    apt-get install -y --fix-broken \
        gcc-arm-linux-gnueabihf libtinfo5 openjdk-11-jdk device-tree-compiler \
        genext2fs libelf-dev autoconf texinfo python2-minimal \
    && rm -rf /var/lib/apt/lists/*

RUN apt-get update --fix-missing && \
    apt-get install -y --fix-broken \
        llvm-15 clang-15 clang-format-15 lld-15 && \
    ln -sf /usr/bin/clang-15 /usr/bin/clang && \
    ln -sf /usr/bin/clang++-15 /usr/bin/clang++ && \
    ln -sf /usr/bin/llvm-ar-15 /usr/bin/llvm-ar && \
    ln -sf /usr/bin/lld-15 /usr/bin/lld && \
    rm -rf /var/lib/apt/lists/*

RUN update-alternatives --install /usr/bin/python python /usr/bin/python3 100 && \
    update-alternatives --install /usr/bin/python2 python2 /usr/bin/python2.7 100 && \
    curl -s https://bootstrap.pypa.io/pip/2.7/get-pip.py | python2 && \
    pip2 install --no-cache-dir --upgrade pip cryptography==3.3.2

RUN pip3 install --no-cache-dir xmltodict cryptography pyyaml asn1crypto

RUN echo "deb http://security.ubuntu.com/ubuntu focal-security main" | tee /etc/apt/sources.list.d/focal-security.list && \
    apt-get update && \
    apt-get install -y libssl1.1 && \
    rm -rf /var/lib/apt/lists/* && \
    rm /etc/apt/sources.list.d/focal-security.list

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        default-jdk u-boot-tools mtools mtd-utils scons gcc-arm-none-eabi \
        android-sdk-libsparse-utils && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        p7zip-full p7zip unzip zip xz-utils \
        dosfstools mtools && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        default-jdk u-boot-tools mtools mtd-utils scons gcc-arm-none-eabi && \
    rm -rf /var/lib/apt/lists/*
```

## Download source code

```bash
$ mkdir cix-ohos && cd cix-ohos
$  ~/bin/repo init -u https://github.com/radxa/orion-oh.git -b orion-oh0-v1.0 -m radxa_release.xml
```

First, pull the HarmonyOS repository

Annotate the GitLab repository in manifests/chipsets/radxa-all.xml

```bash
<!-- <include name="chipsets/cix/radxa_sky1_evb.xml" /> -->
```
then run sync
```bash
$ repo sync -j$(nproc)
```

After downloading the Harmony repository, it is necessary to set up a proxy to improve the speed of downloading the Cix Gitlab repository

Please uncomment GitLab repository in manifests/chipsets/radxa-all.xml
```bash
<include name="chipsets/cix/radxa_sky1_evb.xml" /> 
```

```bash
$ export https_proxy=http://192.168.xxxxx:xxxxx
$ repo sync -j$(nproc)
$ repo forall -c 'git lfs pull'
```

It might take quite a bit of time to fetch the entire source code!

## Build source code

```bash
$ bash build/prebuilts_download.sh
$ ./build.sh --product-name sky1_evb
```

## Generate images

```bash
$ cd vendor/cix/scripts
$ build-storage.sh
```
After executing build-storage.sh, the image is stored in out/images

## Burn BIOS

BIOS download link: https://github.com/radxa/orion-oh/releases/tag/bios-v1.0
1. Copy BIOS to USB flash disk (FAT32)
2. Entering BIOS
3. In the BIOS interface, select Boot Manager --> UEFI Shell to enter the UEFI Shell interface


## Burn image

1. The document author is currently using a Linux environment for burning
2. Enter fastboot mode (directly enter fastboot mode after burning BIOS, or short-circuit the motherboard boot 2 pins)
3. Run fastboot.sh on a Linux terminal until burning is complete



