# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.134.0/containers/cpp/.devcontainer/base.Dockerfile
ARG VARIANT="buster"
FROM mcr.microsoft.com/vscode/devcontainers/cpp:0-${VARIANT}


RUN wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -

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

RUN export DEBIAN_FRONTEND=noninteractive && apt update

# This appears to be necessary to successfully install latest g++ from debian testing ("bullseye")
# See https://www.reddit.com/r/debian/comments/h878ci/fullupgrade_to_debian_testing_fails_due_to/
RUN export DEBIAN_FRONTEND=noninteractive && apt -y install gcc-8-base

# Install latest g++
RUN export DEBIAN_FRONTEND=noninteractive && apt -y install g++

# Install latest clang/llvm/lldb
RUN export DEBIAN_FRONTEND=noninteractive && apt -y install --no-install-recommends \
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
RUN export DEBIAN_FRONTEND=noninteractive && apt -y install gdb

# Upgrade build-essential
RUN export DEBIAN_FRONTEND=noninteractive && apt -y install build-essential

# Install boost
RUN export DEBIAN_FRONTEND=noninteractive && apt -y install libboost-all-dev libboost-tools-dev

# Upgrade cmake
RUN export DEBIAN_FRONTEND=noninteractive && apt -y install cmake

# Cleanup
RUN export DEBIAN_FRONTEND=noninteractive && apt -y remove lldb-7
RUN export DEBIAN_FRONTEND=noninteractive && apt -y autoremove