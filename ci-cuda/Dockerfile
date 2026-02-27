# BOUT++ CUDA dependencies base image
# Provides Spack-managed CUDA toolchain for BOUT-dev CI
# BOUT-dev CI handles cloning/building BOUT++ itself

FROM docker.io/nvidia/cuda:13.1.1-devel-ubuntu24.04 AS base
LABEL maintainer="BOUT++ Development Team"
USER root

# Install system dependencies
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
   ca-certificates \
   libssl-dev &&\
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
 spack add "umpire@2025.12.0+cuda~examples+numa+openmp cuda_arch=${CUDA_ARCH}" &&\
 spack add "raja@2025.12.1+cuda~examples~exercises+openmp cuda_arch=${CUDA_ARCH}" &&\
 spack add fmt@12.1.0 &&\
 spack add 'netcdf-cxx4@4.3.1 ^mpich' &&\
 spack add 'fftw@3.3.10 ^mpich' &&\
 spack external find --not-buildable &&\
 spack external find perl cuda bzip2 pkgconfig ncurses mpich curl openssl --not-buildable &&\
 spack install --show-log-on-error -j$(nproc)

# Set up environment variables for the build
ENV SPACK_ROOT=/spack
ENV PATH="${SPACK_ROOT}/bin:${PATH}"

# Helper script for activating the Spack environment
RUN echo '#!/bin/bash\n\
. /spack/share/spack/setup-env.sh\n\
spack env activate /spack-env\n\
spack load cmake fmt raja umpire netcdf-cxx4 fftw\n\
' > /usr/local/bin/bout-env.bash && \
    chmod +x /usr/local/bin/bout-env.bash

# Install Python packages required by BOUT++
RUN pip3 install --no-cache-dir \
    "Jinja2>=3.1.4" \
    "numpy>=2.0.0" \
    "scipy>=1.14.1" \
    "netcdf4>=1.7.1" \
    "matplotlib>=3.7.0" \
    "Cython>=3.0.0" \
    "boutdata>=0.3.0" \
    "zoidberg>=0.2.2"

# Validate: build BOUT++ (requires --gpus all when running this stage)
# Usage: docker build --target validate --build-arg CUDA_ARCH=<arch> ci-cuda/
FROM setup-spack AS validate

ARG BOUT_URL=https://github.com/boutproject/BOUT-dev.git
ARG BOUT_COMMIT=next
ARG CUDA_ARCH=80

RUN git clone ${BOUT_URL} BOUT-dev && \
    cd BOUT-dev && \
    git checkout ${BOUT_COMMIT} && \
    . /usr/local/bin/bout-env.bash && \
    git submodule update --init --recursive && \
    cmake -S . \
            -B build \
            -DCMAKE_C_COMPILER=gcc \
            -DCMAKE_CXX_COMPILER=g++ \
            -DCMAKE_CUDA_HOST_COMPILER=gcc \
            -DBOUT_ENABLE_RAJA=on \
            -DBOUT_ENABLE_UMPIRE=on \
            -DBOUT_ENABLE_CUDA=on \
            -DCMAKE_CUDA_ARCHITECTURES=${CUDA_ARCH} \
            -DCUDA_ARCH=compute_${CUDA_ARCH},code=sm_${CUDA_ARCH} \
            -DBOUT_ENABLE_WARNINGS=off \
            -DBOUT_USE_SYSTEM_FMT=on && \
      cd build && \
      make -j 2 build-check && \
      make check && cd ../.. && rm -rf BOUT-dev
