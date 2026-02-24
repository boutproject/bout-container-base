# BOUT++ CUDA Base Image

Dependencies-only base container for BOUT-dev CUDA CI.

## Spack environment activation

Inside the container, source the helper script:

```bash
. /usr/local/bin/bout-env.bash
```

## Rebuilding

The image is rebuilt monthly by CI and on every push to `ci-cuda`.
To build locally:

```bash
docker build -t ci-cuda:latest ci-cuda/
```
