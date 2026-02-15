# BOUT++ CUDA Base Image

Dependencies-only base container for BOUT-dev CUDA CI. Provides a Spack-managed
toolchain (CUDA 12.6, RAJA, Umpire, fmt, NetCDF, FFTW) on Ubuntu 22.04.

BOUT-dev CI pulls this image and handles cloning/building BOUT++ itself.

## Image

Published to `ghcr.io/boutproject/bout-container-base:boutdev-cuda`.

## Spack environment activation

Inside the container, source the helper script:

```bash
. /usr/local/bin/bout-env.bash
```

## Rebuilding

The image is rebuilt monthly by CI and on every push to `ci-cuda`.
To build locally:

```bash
docker build -t boutdev-cuda:latest ci-cuda/
```
