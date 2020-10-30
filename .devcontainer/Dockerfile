# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.134.0/containers/cpp/.devcontainer/base.Dockerfile
ARG VARIANT="buster"
FROM mcr.microsoft.com/vscode/devcontainers/cpp:0-${VARIANT}

ARG DEBIAN_FRONTEND=noninteractive

# Avoid apt-key emitting "Warning: apt-key output should not be parsed (stdout is not a terminal)"
ARG APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=1

RUN wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add -

# Add APT sources for debian testing ("bullseye")
RUN echo "\n\
deb http://deb.debian.org/debian testing main contrib non-free\n\
deb-src http://deb.debian.org/debian testing main contrib non-free\n\n\
deb http://deb.debian.org/debian/ testing-updates main contrib non-free\n\
deb-src http://deb.debian.org/debian/ testing-updates main contrib non-free\n\n\
deb http://deb.debian.org/debian-security testing-security main\n\
deb-src http://deb.debian.org/debian-security testing-security main\n\n\
deb http://security.debian.org testing-security main contrib non-free\n\
deb-src http://security.debian.org testing-security main contrib non-free\
" >> /etc/apt/sources.list

# Add APT sources for latest clang/llvm
ARG LLVM_VERSION=11

RUN echo "\n\
deb http://apt.llvm.org/unstable/ llvm-toolchain-${LLVM_VERSION} main\n\
deb-src http://apt.llvm.org/unstable/ llvm-toolchain-${LLVM_VERSION} main\
" >> /etc/apt/sources.list

RUN apt-get update &&\
    apt-get -y -f -o APT::Immediate-Configure=0 install gcc-8-base g++

# Install latest clang/llvm/lldb
RUN apt-get -y install --no-install-recommends \
    llvm-${LLVM_VERSION} \
    clang-${LLVM_VERSION} \
    clang-tools-${LLVM_VERSION} \
    clang-${LLVM_VERSION}-doc \
    libclang-common-${LLVM_VERSION}-dev \
    libclang-${LLVM_VERSION}-dev \
    libclang1-${LLVM_VERSION} \
    clang-format-${LLVM_VERSION} \
    clangd-${LLVM_VERSION} \
    lldb-${LLVM_VERSION} \
    lld-${LLVM_VERSION} \
    libc++-${LLVM_VERSION}-dev \
    libc++abi-${LLVM_VERSION}-dev \
    libomp-${LLVM_VERSION}-dev 

# Upgrade gdb
RUN apt-get -y install \
    gdb \
    build-essential \
    libboost-all-dev \
    libboost-tools-dev \
    cmake

# Cleanup
RUN apt-get -y remove lldb-7 &&\
    apt-get -y autoremove

