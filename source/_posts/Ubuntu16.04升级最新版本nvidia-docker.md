title: Ubuntu 16.04 升级最新版本 nvidia-docker

tags:
- ubuntu
- nvidia-docker

categories:
- Linux
- docker

date: 2019/12/09 17:18:48

---

@[TOC]

### Uninstall old versions

Older versions of Docker were called `docker`, `docker.io `, or `docker-engine`. If these are installed, uninstall them:

```
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

It’s OK if `apt-get` reports that none of these packages are installed.

The contents of `/var/lib/docker/`, including images, containers, volumes, and networks, are preserved. The Docker Engine - Community package is now called `docker-ce`.

### Supported storage drivers

Docker Engine - Community on Ubuntu supports `overlay2`, `aufs` and `btrfs` storage drivers.

> **Note**: In Docker Engine - Enterprise, `btrfs` is only supported on SLES. See the documentation on [btrfs](https://docs.docker.com/engine/userguide/storagedriver/btrfs-driver/) for more details.

For new installations on version 4 and higher of the Linux kernel, `overlay2` is supported and preferred over `aufs`. Docker Engine - Community uses the `overlay2` storage driver by default. If you need to use `aufs` instead, you need to configure it manually. See [aufs](https://docs.docker.com/engine/userguide/storagedriver/aufs-driver/)

### Install Docker Engine - Community

You can install Docker Engine - Community in different ways, depending on your needs:

- Most users [set up Docker’s repositories](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-using-the-repository) and install from them, for ease of installation and upgrade tasks. This is the recommended approach.
- Some users download the DEB package and [install it manually](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-from-a-package) and manage upgrades completely manually. This is useful in situations such as installing Docker on air-gapped systems with no access to the internet.
- In testing and development environments, some users choose to use automated [convenience scripts](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-using-the-convenience-script) to install Docker.

### Install using the repository

Before you install Docker Engine - Community for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

#### SET UP THE REPOSITORY

1. Update the `apt` package index:

   ```
   $ sudo apt-get update
   ```

2. Install packages to allow `apt` to use a repository over HTTPS:

   ```
   $ sudo apt-get install \
       apt-transport-https \
       ca-certificates \
       curl \
       gnupg-agent \
       software-properties-common
   ```

3. Add Docker’s official GPG key:

   ```
   $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```

   Verify that you now have the key with the fingerprint `9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88`, by searching for the last 8 characters of the fingerprint.

   ```
   $ sudo apt-key fingerprint 0EBFCD88

   pub   rsa4096 2017-02-22 [SCEA]
         9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
   uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
   sub   rsa4096 2017-02-22 [S]
   ```

4. Use the following command to set up the **stable** repository. To add the **nightly** or **test** repository, add the word `nightly` or `test` (or both) after the word `stable` in the commands below. [Learn about **nightly** and **test** channels](https://docs.docker.com/install/).

   > **Note**: The `lsb_release -cs` sub-command below returns the name of your Ubuntu distribution, such as `xenial`. Sometimes, in a distribution like Linux Mint, you might need to change `$(lsb_release -cs)` to your parent Ubuntu distribution. For example, if you are using `Linux Mint Tessa`, you could use `bionic`. Docker does not offer any guarantees on untested and unsupported Ubuntu distributions.



   - - x86_64 / amd64
     - armhf
     - arm64
     - ppc64le (IBM Power)
     - s390x (IBM Z)



   ```
   $ sudo add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
   ```



#### INSTALL DOCKER ENGINE - COMMUNITY

1. Update the `apt` package index.

   ```
   $ sudo apt-get update
   ```

2. Install the *latest version* of Docker Engine - Community and containerd, or go to the next step to install a specific version:

   ```
   $ sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```

   > **** Got multiple Docker repositories?
   >
   > If you have multiple Docker repositories enabled, installing or updating without specifying a version in the `apt-get install` or `apt-get update` command always installs the highest possible version, which may not be appropriate for your stability needs.

3. To install a *specific version* of Docker Engine - Community, list the available versions in the repo, then select and install:

   a. List the versions available in your repo:

   ```
   $ apt-cache madison docker-ce

     docker-ce | 5:18.09.1~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
     docker-ce | 5:18.09.0~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
     docker-ce | 18.06.1~ce~3-0~ubuntu       | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
     docker-ce | 18.06.0~ce~3-0~ubuntu       | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
     ...
   ```

   b. Install a specific version using the version string from the second column, for example, `5:18.09.1~3-0~ubuntu-xenial`.

   ```
   $ sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
   ```

4. Verify that Docker Engine - Community is installed correctly by running the `hello-world` image.

   ```shell
   $ sudo docker run hello-world
   ```