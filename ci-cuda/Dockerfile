# BOUT++ CUDA-enabled Docker container
# This Dockerfile builds BOUT++ with CUDA support using Spack for dependency management

FROM docker.io/nvidia/cuda:12.6.1-devel-ubuntu22.04 AS base
LABEL maintainer="BOUT++ Development Team"
USER root

# Install system dependencies
# Using Ubuntu 22.04 for more recent toolchain and better CUDA support
RUN \
 apt update &&\
 DEBIAN_FRONTEND=noninteractive TZ=America/Los_Angeles apt install -y \
   build-essential \
   git \
   software-properties-common \
   python3 \
   python3-pip \
   wget \
   mpich \
   libmpich-dev \
   unzip \
   libnuma-dev \
   libncurses-dev \
   curl \
   libcurl4-openssl-dev \
   gfortran \
   ca-certificates &&\
 apt upgrade -y &&\
 apt clean

FROM base AS setup-spack

# CUDA architecture: 70=Volta, 75=Turing, 80=Ampere, 89=Ada, 90=Hopper
ARG CUDA_ARCH=80

# Install Spack and set up environment
RUN \
 git clone --depth 1 --single-branch --branch v0.23.0 https://github.com/spack/spack.git &&\
 . /spack/share/spack/setup-env.sh &&\
 spack env create -d /spack-env &&\
 spack env activate /spack-env &&\
 spack add cmake@3.24: &&\
 spack add blt@0.6.2 &&\
 spack add "umpire@2024.07.0+cuda~examples+numa+openmp cuda_arch=${CUDA_ARCH}" &&\
 spack add "raja@2024.07.0+cuda~examples~exercises+openmp cuda_arch=${CUDA_ARCH}" &&\
 spack add fmt@11.0.2 &&\
 spack add netcdf-cxx4@4.3.1 &&\
 spack add fftw@3.3.10 &&\
 spack external find --not-buildable &&\
 spack external find perl cuda bzip2 pkgconfig ncurses mpich curl --not-buildable &&\
 spack install -j$(nproc)

# Set up environment variables for the build
ENV SPACK_ROOT=/spack
ENV PATH="${SPACK_ROOT}/bin:${PATH}"

FROM setup-spack AS bout-build

ARG CUDA_ARCH=80

# Create boutuser (for consistency with existing Docker setup)
RUN useradd -m -s /bin/bash boutuser && \
    echo "boutuser:boutforever" | chpasswd && \
    usermod -aG sudo boutuser

USER boutuser
WORKDIR /home/boutuser

ARG BOUT_URL=https://github.com/boutproject/BOUT-dev.git
ARG BOUT_COMMIT=next

WORKDIR /home/boutuser

# Activate Spack environment and configure BOUT++ with CUDA
USER root
RUN . /spack/share/spack/setup-env.sh && \
    spack env activate /spack-env && \
    spack load cmake && \
    spack load fmt && \
    spack load raja && \
    spack load umpire && \
    spack load netcdf-cxx4 && \
    spack load fftw && \
    chown -R boutuser:boutuser /home/boutuser

RUN echo '# Setup spack for BOUT++\n\
. /spack/share/spack/setup-env.sh\n\
spack env activate /spack-env\n\
spack load cmake fmt raja umpire netcdf-cxx4 fftw\n\
' > /usr/local/bin/bout-env.bash

USER boutuser

RUN git clone ${BOUT_URL} BOUT-dev && \
    cd BOUT-dev && \
    git checkout ${BOUT_COMMIT} && \
    . /usr/local/bin/bout-env.bash && \
    git submodule update --init --recursive && \
    cmake -S . \
            -B build \
            -DCMAKE_C_COMPILER=gcc \
            -DCMAKE_CXX_COMPILER=g++ \
            -DBOUT_ENABLE_RAJA=on \
            -DBOUT_ENABLE_UMPIRE=on \
            -DBOUT_ENABLE_CUDA=on \
            -DCMAKE_CUDA_ARCHITECTURES=80 \
            -DCUDA_ARCH=compute_80,code=sm_80 \
            -DBOUT_ENABLE_WARNINGS=off \
            -DBOUT_USE_SYSTEM_FMT=on && \
      cd build && \
      make -j 2 build-check && \
      make check && cd ../.. && rm -rf BOUT-dev
