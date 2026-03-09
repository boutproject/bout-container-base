FROM registry.fedoraproject.org/fedora:rawhide
LABEL maintainer="BOUT++ Development Team"

RUN echo "install_weak_deps=False" >> /etc/dnf/dnf.conf && \
    echo "minrate=10M" >> /etc/dnf/dnf.conf && \
    export FORCE_COLUMNS=200 && \
    rpm -q dnf5 || (time dnf -y install dnf5 ; time dnf clean all ) ; \
    time dnf5 -y install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm && \
    time dnf5 -y config-manager addrepo --from-repofile=https://developer.download.nvidia.com/compute/cuda/repos/fedora42/$(uname -m)/cuda-fedora42.repo && \
    time dnf5 -y config-manager setopt cuda-fedora42-$(uname -m).exclude=nvidia-driver,nvidia-modprobe,nvidia-persistenced,nvidia-settings,nvidia-libXNVCtrl,nvidia-xconfig && \
    time dnf5 -y install cuda-toolkit xorg-x11-drv-nvidia-cuda gcc14-c++ && \
    time dnf5 -y install dnf5-plugins cmake python3-zoidberg python3-natsort python3-boututils git-core wget && \
    time dnf5 copr enable -y davidsch/fixes4bout ; \
    time dnf5 -y upgrade && \
    time dnf5 -y builddep bout++ && \
    time dnf5 -y install cpptrace-devel && \
    time dnf5 clean all

RUN useradd test -G wheel -p '$6$MoHfQDiMU5ajgDMm$9FAMLxMflKwQCZ.sJBNG6wLGnPeySVizdA8wN0k8LSXKPkCfOb/sM9Y4jKFvh5rzKSwLtYSTzvJyETqrFlxBV.' \
    && echo '%wheel ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

ARG BOUT_URL=https://github.com/boutproject/BOUT-dev.git
ARG BOUT_COMMIT=c++20
ARG CUDA_ARCH=80

RUN git clone ${BOUT_URL} BOUT-dev && \
    cd BOUT-dev && \
    git checkout ${BOUT_COMMIT} && \
    git submodule update --init --recursive --recommend-shallow && \
    . /etc/profile.d/modules.sh && \
    module load mpi/openmpi-x86_64 && \
    export OMPI_MCA_rmaps_base_oversubscribe=yes && \
    export PRTE_MCA_rmaps_default_mapping_policy=:oversubscribe && \
    export TRAVIS=true && \
    export FLEXIBLAS=NETLIB && \
    export MKL_NUM_THREADS=1 && \
    export NUMEXPR_NUM_THREADS=1 && \
    export OMP_NUM_THREADS=1 && \
    export PATH=/usr/local/cuda-13.1/bin:$PATH && \
    cmake -S . \
            -B build \
            -DCMAKE_C_COMPILER=gcc-14 \
            -DCMAKE_CXX_COMPILER=g++-14 \
            -DCMAKE_CUDA_HOST_COMPILER=gcc-14 \
            -DBOUT_ENABLE_RAJA=on \
            -DBOUT_ENABLE_UMPIRE=on \
            -DBOUT_ENABLE_CUDA=on \
            -DCMAKE_CUDA_COMPILER=/usr/local/cuda-13.1/bin/nvcc \
            -DCMAKE_CUDA_ARCHITECTURES=${CUDA_ARCH} \
            -DCUDA_ARCH=compute_${CUDA_ARCH},code=sm_${CUDA_ARCH} \
            -DBOUT_ENABLE_WARNINGS=off \
            -DBOUT_USE_SYSTEM_FMT=on && \
      cd build && \
      make -j 2 build-check && \
      make check && cd ../.. && rm -rf BOUT-dev
