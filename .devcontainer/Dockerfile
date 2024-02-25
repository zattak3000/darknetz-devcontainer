FROM mcr.microsoft.com/devcontainers/base:jammy

ARG DEBIAN_FRONTEND=noninteractive

RUN apt update && apt upgrade -y
RUN apt install -y \
    adb \
    acpica-tools \
    autoconf \
    automake \
    bc \
    bison \
    build-essential \
    ccache \
    cpio \
    cscope \
    curl \
    device-tree-compiler \
    dosfstools \
    e2tools \
    expect \
    fastboot \
    fdisk \
    flex \
    ftp-upload \
    gdisk \
    git \
    libattr1-dev \
    libcap-ng-dev \
    libfdt-dev \
    libftdi-dev \
    libglib2.0-dev \
    libgmp3-dev \
    libhidapi-dev \
    libmpc-dev \
    libncurses5-dev \
    libpixman-1-dev \
    libslirp-dev \
    libssl-dev \
    libtool \
    libusb-1.0-0-dev \
    make \
    mtools \
    netcat \
    ninja-build \
    parted \
    python3-cryptography \
    python3-pip \
    python3-pyelftools \
    python3-serial \
    python-is-python3 \
    rsync \
    swig \
    unzip \
    uuid-dev \
    wget \
    xdg-utils \
    xterm \
    xz-utils \
    zlib1g-dev

WORKDIR /optee
RUN git clone https://github.com/OP-TEE/build
RUN git clone https://github.com/OP-TEE/optee_os
RUN git clone https://github.com/OP-TEE/optee_client

WORKDIR /optee/build
RUN make -f rpi3.mk -j2 toolchains

ENV PATH="${PATH}:/optee/toolchains/aarch64/bin:/optee/toolchains/aarch32/bin"

WORKDIR /optee/optee_client
RUN make CROSS_COMPILE=aarch64-linux-gnu- WITH_TEEACL=0

WORKDIR /optee/optee_os
RUN make \
    CFG_ARM64_core=y \
    CFG_TEE_BENCHMARK=n \
    CFG_TEE_CORE_LOG_LEVEL=3 \
    CROSS_COMPILE=aarch64-linux-gnu- \
    CROSS_COMPILE_core=aarch64-linux-gnu- \
    CROSS_COMPILE_ta_arm32=arm-linux-gnueabihf- \
    CROSS_COMPILE_ta_arm64=aarch64-linux-gnu- \
    DEBUG=1 \
    O=out/arm \
    PLATFORM=rpi3

ENV CROSS_COMPILE=aarch64-linux-gnu-
ENV PLATFORM=rpi3
ENV TA_DEV_KIT_DIR=/optee/optee_os/out/arm/export-ta_arm64
ENV TEEC_EXPORT=/optee/optee_client/out/export/usr

WORKDIR /optee/darknetz
# RUN git clone https://github.com/mofanv/darknetz.git .

ENTRYPOINT [ "make" ]