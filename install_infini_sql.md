
# 安装 Infini SQL 引擎（ MegaWise ）

本文档主要介绍 MegaWise Docker 容器镜像的安装和配置等操作，完成后即可连接 MegaWise 进行各类数据操作。文档中涉及的操作主要包含以下部分：

- **安装前提**。该部分用于安装和配置 MegaWise Docker 镜像的运行环境，主要包括了 NVIDIA 驱动、Docker 和 NVIDIA container toolkit 的安装。
- **MegaWise Docker 镜像的获取和配置**。该部分包括 Docker 镜像的获取、初始化设置、启动 MegaWise、修改 PostgreSQL 相关参数等。
- **安装验证**。启动容器并检查容器的运行情况和配置的正确性。

## 安装前提

- 运行 MegaWise Docker 镜像要求服务器的操作系统为 Ubuntu 16.04 及以上版本。
- 禁用 Nouveau 驱动，并安装 NVIDIA driver 430，具体参照 [安装 NVIDIA 驱动](#安装-NVIDIA-驱动) 。如已安装，请使用 `nvidia-smi` 命令检查是否安装成功。
- [安装 Docker](#安装-Docker)，版本不低于1.12，建议安装 Docker 19.03。如已安装，请使用 `docker -v` 命令检查。
- [安装NVIDIA container toolkit](#安装-NVIDIA-container-toolkit)。

### 安装 NVIDIA 驱动

1. 禁用 Nouveau 驱动。

   安装 NVIDIA 驱动之前必须先禁用 Nouveau 驱动。请通过以下命令检查是否已启用 Nouveau 驱动：

   ```
   $ lsmod | grep nouveau  
   ```

   该命令执行后，如果终端打印了相关信息则说明 Nouveau 驱动已经被启用。如果启用了 Nouveau 驱动，则需执行后续的步骤将其禁用，否则请跳过以下步骤，开始安装 NVIDIA 驱动。

   在以下路径创建文件 `/etc/modprobe.d/blacklist-nouveau.conf` 并在文件中写入如下内容：

   ```
   blacklist nouveau
   options nouveau modeset=0  
   ```

   执行以下命令并重启系统：

   ```
   $ sudo update-initramfs -u
   $ sudo reboot  
   ```

2. 使用 apt 工具安装 NVIDIA 驱动

   > 注意：Megawies 当前仅支持430版本的 NVIDIA 驱动。

   ```
   sudo add-apt-repository ppa:graphics-drivers/ppa
   sudo apt update
   sudo apt search nvidia-*
   sudo apt install nvidia-430  
   ```

3. 重启 NVIDIA 驱动：

   ```
   sudo reboot  
   ```

4. 测试是否安装成功：

   ```
   $ sudo nvidia-smi  
   ```

   如果安装正确，终端打印的内容将包含类似如下信息：

   ```
   +-----------------------------------------------------------------------------+
   | NVIDIA-SMI 430.34       Driver Version: 430.34       CUDA Version: 10.1     |
   |-------------------------------+----------------------+----------------------+
   | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
   | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
   |===============================+======================+======================|
   |   0  GeForce GTX 1660    Off  | 00000000:01:00.0  On |                  N/A |
   | 28%   49C    P0    24W / 130W |   2731MiB /  5941MiB |      1%      Default |
   +-------------------------------+----------------------+----------------------+
   ```
### 安装 Docker

安装 Docker 之前，请运行以下命令检查系统中是否已经安装过 Docker。

```
docker -v
```

如果终端能够显示 Docker 的版本信息，则说明系统中已经安装有对应版本的 Docker。记录下系统中的Docker 版本，后续的操作中需要用到。在如下的示例中 Docker 的版本为18.09.7。
```
Docker version 18.09.7, build 2d0083d
```
如果系统中已安装有 Docker 且 Docker 版本不低于1.12，则可跳过 Docker 安装步骤，开始 [安装 NVIDIA container toolkit](#安装-NVIDIA-container-toolkit)。如果系统中未安装 Docker，则参照以下步骤开始安装 Docker。

> 注意：MegaWise Docker 镜像不支持1.12以下版本的Docker环境。

1. 更新 apt-get。

   ```
   sudo apt-get update
   ```

2. 使用 curl 工具下载最新版本的 Docker。

   ```
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   Add Docker to your Apt repository.
   sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   ```

   如果系统中未安装 curl 工具，则先使用 apt-get 安装 curl ,然后执行上述命令。

   ```
   sudo apt-get install curl
   ```

3. 更新 apt-get 仓库。

   ```
   sudo apt-get update
   ```

4. 安装 Docker 及其相关的命令行接口和 runtime 环境。

   ```
   sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```

5. 重新执行以下命令验证 Docker 是否安装成功。如果能够打印 Docker 的版本信息，则说明已成功安装Docker。

   ```
   docker -v
   ```

### 安装 NVIDIA container toolkit

1. 使用 curl 添加 gpg key。

   ```
   curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
   sudo apt-key add -
   distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
   ```

2. 更新下载源。

   ```
   curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
   sudo tee /etc/apt/sources.list.d/nvidia-docker.list
   ```

3. 根据之前获取的 Docker 版本，使用 apt-get 安装 NVIDIA runtime。如果 Docker 为19.03或者更高的版本，则安装 nvidia-container-toolkit。

   ```
   sudo apt-get update
   sudo apt-get install -y nvidia-container-toolkit
   ```

   如果 Docker 版本低于19.03，则使用以下命令安装 nvidia-docker2。

   ```
   sudo apt-get update
   sudo apt-get install -y nvidia-docker2
   ```

4. 重启 Docker daemon

   ```
   sudo systemctl restart docker
   ```

5. 验证 NVIDIA container toolkit 是否安装成功。

   ```
   docker run --gpus all nvidia/cuda:9.0-base nvidia-smi 
   ```

   如果 Docker 版本低于19.03，则执行以下命令。

   ```
   sudo docker run --runtime=nvidia --rm nvidia/cuda:9.0-base nvidia-smi
   ```

如果能够成功打印服务器 GPU 信息，则表示安装成功。

## 开始安装 Infini SQL 引擎（ MegaWise ）

1. 请确保当前用户对目录 `/tmp` 有读写权限

2. 下载脚本 `install.sh` 和 `data_import.sh` 至同一目录，并确保当前用户对两个脚本有可执行权限

   ```bash
   $ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/script/data_import.sh \
   https://raw.githubusercontent.com/Infini-Analytics/infini/master/script/install_megawise.sh
   $ chmod a+x *.sh
   ```
   
3. 安装MegaWise并导入示例数据

   ```bash
   $ ./install_megawise.sh [参数1，必选] [参数2，可选]
   ```

   > 参数1：MegaWise安装目录的地址，
   >
   > 参数2：MegaWise镜像id，可选，默认'0.3.0-d091919-1679'
   >
   > 示例：./install_megawise.sh  /home/$USER/megawise '0.3.0-d091919-1679'
