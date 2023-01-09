# Install Singularity

Instructions are based on [this guide](https://singularity-tutorial.github.io/01-installation/).

1. Install prerequisites

   ```sh
   sudo apt update && sudo apt-get install -y build-essential libssl-dev uuid-dev libgpgme11-dev \
       squashfs-tools libseccomp-dev wget pkg-config git cryptsetup debootstrap
   ```

2. Install GO

   ```sh
   wget https://dl.google.com/go/go1.13.linux-amd64.tar.gz 
   sudo tar --directory=/usr/local -xzvf go1.13.linux-amd64.tar.gz
   export PATH=/usr/local/go/bin:$PATH
   rm go1.13.linux-amd64.tar.gz
   ```

   <!-- **Reload the shell** by typing in `bash` or `zsh`, based on your shell. -->

3. Download Singularity

   ```sh
   mkdir -p ~/singularity && cd ~/singularity
   wget https://github.com/sylabs/singularity/releases/download/v3.10.4/singularity-ce-3.10.4.tar.gz
   tar -xzvf singularity-ce-3.10.4.tar.gz && rm singularity-ce-3.10.4.tar.gz
   ```

4. Install Singularity

   ```sh
   cd singularity-ce-3.10.4
   ./mconfig
   cd builddir
   make
   sudo make install
   ```