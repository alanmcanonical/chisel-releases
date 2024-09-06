# ubuntu-22.04

This is the `ubuntu-22.04` Chisel release with PoC support for FIPS.

To learn more about Chisel, Chisel releases and how to contribute to this
branch, please read
[this](https://github.com/canonical/chisel-releases/blob/main/README.md).

## Prerequisites

Before you begin, ensure you have the following installed:


1. **Go**: You need to have Go installed on your system. You can install it using one of the following methods:

   - **Download from the Official Site**:
     - Download the latest version of Go from [https://go.dev/dl](https://go.dev/dl).
     - Extract the downloaded tarball:

       ```bash
       tar -C ~/dev -xf https://go.dev/dl/go1.XX.linux-amd64.tar.gz  # Replace with the actual filename
       ```

     - Add Go to your PATH by adding the following line to your `~/.bashrc`:

       ```bash
       export PATH=$PATH:~/dev/go/bin
       ```

     - Apply the changes:

       ```bash
       source ~/.bashrc
       ```

     - Check the go installation:

       ```bash
       go version
       ```

   - **Install via APT**:
     - You can also install Go using the package manager:

       ```bash
       sudo apt update
       sudo apt install golang
       ```

2. **LXD**: If LXD is not installed, run the following command:

   ```bash
   snap install lxd
   ```

   - If LXD is not running, you can start it with:

   ```bash
   lxd init --minimal  # Drop the --minimal for an interactive configuration
   ```

3. **Rockcraft**: Install Rockcraft, if you want to install the custom package slice into a rock though:
    
    ```bash
    sudo snap install rockcraft --classic

    # Testing Rockcraft
    rockcraft --version
    ```

4. **Docker**: If you want to use Rockcraft:

    ```bash
    # Add Docker's official GPG key:
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc

    # Add the repository to Apt sources:
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update

    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

## Build Process

Follow these steps to build the Chisel release with FIPS support:

- Clone the Chisel Repository and checkout the pro-archives branch:

```bash
git clone https://github.com/rebornplusplus/chisel.git
cd chisel
git checkout feat/pro-archives
```

- (optional) Run the following command to generate the version file:

```bash
go generate ./cmd/
```

- Use the following command to build Chisel:

```bash
go build -trimpath -ldflags='-s -w' ./cmd/chisel
```

- Clone the Chisel release repository and checkout the FIPS branch:

```bash
git clone https://github.com/alanmcanonical/chisel-releases.git
cd chisel-releases
git checkout 22.04/fips
```

- (optional) Check Custom ubuntu-advantage-tools Slice:

```bash
chisel find --release=./chisel-releases/ ubuntu-advantage-tools_bins
```

- Initialize a New Rockcraft Project:

```bash
rockcraft init
```

After this command, you should find a new rockcraft.yaml file in your current path.

- Adjust the rockcraft.yaml File:

```yaml
name: custom-uat-rock
base: bare
build_base: "ubuntu@22.04"
version: '0.0.1'
summary: A chiselled rock with a custom ubuntu-advantage-tools slice
description: |
    A rock containing only the binaries (and corresponding dependencies) from the Ubuntu-advantage-tools package.
    Built from a custom Chisel release.
license: GPL-3.0
platforms:
    amd64:
parts:
    build-context:
        plugin: nil
        source: chisel-releases/
        source-type: local
        stage-packages:
        - bash_bins
        - ca-certificates_bins
        - ca-certificates_data-with-certs
        override-build:
            chisel cut --release ./ --root ${CRAFT_PART_INSTALL} ubuntu-advantage-tools_bins
```

Build Your Rock:

```bash
rockcraft pack
```

Test the ubuntu-advantage-tools Binaries:

```bash
sudo rockcraft.skopeo --insecure-policy copy oci-archive:custom-openssl-rock_0.0.1_amd64.rock docker-daemon:chisel-fips:latest
```

And after:

```bash
docker run --name myfips -d chisel-fips:latest
docker -exec -it myfips /bin/bash

# Check pro status
pro status
# Attach pro
pro attach your_token
pro enable fips-preview
```

