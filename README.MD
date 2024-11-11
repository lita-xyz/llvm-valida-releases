See 'Releases' section for downloads.

# Docker Support

We provide a Docker container with the Valida LLVM and Rust toolchains already installed.
Docker is the only supported method of running on platforms other than x86 Linux.

```bash
# Download the container
docker pull ghcr.io/lita-xyz/llvm-valida-releases/valida-build-container:v0.5.0-alpha

cd your-valida-project

# Enter the container:
docker run --platform linux/amd64 --entrypoint=/bin/bash -it --rm -v $(realpath .):/src ghcr.io/lita-xyz/llvm-valida-releases/valida-build-container:v0.5.0-alpha

# You are now in a shell with the valida rust toolchain installed!
```