# Install Singularity

Instructions are based on [the official guide](https://github.com/sylabs/singularity/blob/main/INSTALL.md).

1. Install prerequisites

   ```sh
   sudo apt-get update && sudo apt-get install -y build-essential \
    libseccomp-dev libglib2.0-dev pkg-config squashfs-tools \
    cryptsetup crun uidmap
   ```

2. Install GO

   ```sh
   export VERSION=1.19.5 OS=linux ARCH=amd64  # change this as you need

   wget -O /tmp/go${VERSION}.${OS}-${ARCH}.tar.gz \
   https://dl.google.com/go/go${VERSION}.${OS}-${ARCH}.tar.gz
   sudo tar -C /usr/local -xzf /tmp/go${VERSION}.${OS}-${ARCH}.tar.gz
   export PATH=$PATH:/usr/local/go/bin
   ```

   <!-- **Reload the shell** by typing in `bash` or `zsh`, based on your shell. -->

3. Download Singularity

   ```sh
   git clone --recurse-submodules https://github.com/sylabs/singularity.git
   cd singularity
   ```

4. Install Singularity

   ```sh
   ./mconfig
   make -C builddir
   sudo make -C builddir install
   ```