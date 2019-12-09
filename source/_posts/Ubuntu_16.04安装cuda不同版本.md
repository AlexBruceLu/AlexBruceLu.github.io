title: Ubuntu 16.04 安装 CUDA

tags:

- ubuntu
- CUDA

categories:

- Linux
- NVIDIA

date: 2019/12/09 19:05:23

------

@[TOC]

## **一、操作系统:**

研发工程师默认都是台式机，安装ubuntu16.04操作系统，默认我们安装好cuda和显卡驱动相关软件。如果需要重装系统请联系各属地IT，目前采用USB自动安装的方式，10分钟内可以安装完成

## **二、卸载原有的驱动和cuda:**

sudo su root

service lightdm stop

nvidia-uninstall

apt-get install autoremove --purge nvidia*

/usr/local/cuda-8.0/bin/uninstall_cuda-8.0.pl(根据版本号有差别，此处以8.0为例子)

rm -rf /usr/local/cuda-8.0(!!!执行这一步时切勿误操作，否则容易导致系统崩溃)

## **三、cuda安装：包含显卡驱动和相关软件包**

### 1.下载cuda软件

**（8.0版本）** wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/local_installers/cuda_8.0.61_375.26_linux-run

**（9.0版本）** wget https://developer.nvidia.com/compute/cuda/9.0/Prod/local_installers/cuda_9.0.176_384.81_linux-run

**（10.0版本）**wget https://developer.nvidia.com/compute/cuda/10.0/Prod/local_installers/cuda_10.0.130_410.48_linux

**（10.1版本）** 

​		wget http://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.243_418.87.00_linux.run

**（10.2版本）**

​		 wget http://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda_10.2.89_440.33.01_linux.run

### 2.下载NVIDIA显卡驱动

wget http://cn.download.nvidia.com/XFree86/Linux-x86_64/440.36/NVIDIA-Linux-x86_64-440.36.run

### 3.添加安装程序的可执行权限：

chmod +x *.run

### 4.禁用默认的nouveau开源显卡驱动：

echo "blacklist nouveau" >> /etc/modprobe.d/blacklist-nouveau.conf

之后建议重启下系统

### 5.关闭系统图形界面，显卡驱动安装过程需要在命令行模式下进行（Ctrl + Alt + F2 进入命令行模式）

```
sudo su root

service lightdm stop
```

### 6.安装显卡驱动程序，命令：

```
./NVIDIA-Linux-x86_64-410.78.run -a -s -Z --no-opengl-files
```

### 7.安装CUDA-9.0 驱动（请把cuda安装程序和安装目标目录替换为自己需要安装的对应版本）

```
./cuda_9.0.176_384.81_linux.run --no-opengl-libs --toolkit --samples --samplespath=/usr/local/cuda-9.0 -silent
```

### 8.添加安装的环境变量（将红色部分替换为自己安装的对应版本）:

```
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib:/usr/local/cuda/lib64/:/usr/local/cuda-9.0/lib:/usr/local/cuda-9.0/lib64/:$LD_LIBRARY_PATH' >> /etc/profile
echo 'export PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/cuda-9.0/bin:/usr/local/cuda/bin:$PATH' >> /etc/profile
echo '/usr/local/cuda/lib64' >> /etc/ld.so.conf.d/cuda.conf

ldconfig
```

### 9.下载cudnn

- **Download cuDNN v7.6.5 (November 18th, 2019), for CUDA 10.2**

  ```bash
   wget https://developer.nvidia.com/compute/machine-learning/cudnn/secure/7.6.5.32/Production/10.2_20191118/cudnn-10.2-linux-x64-v7.6.5.32.tgz
  ```

- **Download cuDNN v7.6.5 (November 5th, 2019), for CUDA 10.1**

  ```
  wget https://developer.nvidia.com/compute/machine-learning/cudnn/secure/7.6.5.32/Production/10.1_20191031/cudnn-10.1-linux-x64-v7.6.5.32.tgz
  ```

- **Download cuDNN v7.6.4 (September 27, 2019), for CUDA 10.0**

  ```
  wget https://developer.nvidia.com/compute/machine-learning/cudnn/secure/7.6.4.38/Production/10.0_20190923/cudnn-10.0-linux-x64-v7.6.4.38.tgz
  ```

- **Download cuDNN v7.6.3 (August 23, 2019), for CUDA 9.0**

  ```
  wget https://developer.nvidia.com/compute/machine-learning/cudnn/secure/7.6.3.30/Production/9.0_20190822/cudnn-9.0-linux-x64-v7.6.3.30.tgz
  ```

- **Download cuDNN v7.1.3 (April 17, 2018), for CUDA 8.0**

  ```
  wget https://developer.nvidia.com/compute/machine-learning/cudnn/secure/v7.1.3/prod/8.0_20180414/cudnn-8.0-linux-x64-v7.1
  ```

### 10.安装cudnn

```
tar xf cudnn-9.0-linux-x64-v7.tar
mv cuda/lib64/libcudnn* /usr/local/cuda-9.0/lib64/
mv cuda/include/cudnn.h /usr/local/cuda-9.0/include/
```