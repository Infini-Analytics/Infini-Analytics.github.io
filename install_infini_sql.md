
# 安装 Infini SQL 引擎（ MegaWise ）

本文档主要介绍 MegaWise Docker 的安装和配置等操作，完成后即可连接 MegaWise 进行各类数据操作。文档中涉及的操作主要包含以下部分：

- **安装前提** &ndash;包括硬件要求和软件要求，以及如何安装和配置 MegaWise Docker 的软件运行环境。主要涵盖了 NVIDIA 驱动、Docker 和 NVIDIA container toolkit 的安装。
- **安装 MegaWise Docker 并导入示例数据** &ndash;包括 MegaWise Docker 的安装以及 infini 示例数据的导入。



## 安装前提

### 硬件要求

**!!!需要提供硬件要求信息!!!**

### 软件要求

| 组件                     | 版本                    |
|--------------------------|-------------------------|
| 操作系统                 | Ubuntu 16.04 或以上 |
| NVIDIA 驱动          | 430或以上           |
| Docker                   | 19.03或以上         |
| NVIDIA Container Toolkit |  **!!!需要提供版本信息!!!**                      |


### 安装 NVIDIA 驱动


1. 禁用 Nouveau 驱动

   安装 NVIDIA 驱动之前必须先禁用 Nouveau 驱动。请通过以下命令检查是否已启用 Nouveau 驱动：

   ```bash
   $ lsmod | grep nouveau  
   ```

   该命令执行后，如果终端打印了相关信息则说明 Nouveau 驱动已经被启用。如果启用了 Nouveau 驱动，则需执行后续的步骤将其禁用，否则请跳过以下步骤，开始安装 NVIDIA 驱动。

   在以下路径创建文件 `/etc/modprobe.d/blacklist-nouveau.conf` 并在文件中写入如下内容：

   ```
   blacklist nouveau
   options nouveau modeset=0  
   ```

   执行以下命令并重启系统：

   ```bash
   $ sudo update-initramfs -u
   $ sudo reboot  
   ```

   确认禁用 Nouveau 驱动，执行该命令将不输出任何信息。

   ```bash
   $ lsmod | grep nouveau
   ```
   
   如果系统中未安装 lsmod 工具，则先安装 lsmod, 然后执行上述命令。

   ```bash
   $ sudo apt-get install lsmod
   ```

2. 从 [NVIDIA官方驱动下载链接](https://www.nvidia.com/Download/index.aspx?lang=en-us) 下载驱动安装文件。

   > <font color='red'>注意：MegaWise 当前仅支持430及以上版本的 NVIDIA 驱动。安装或更新 NVIDIA 驱动存在一定风险，有可能导致显示系统崩溃。在安装或更新 NVIDIA 驱动前，请在[NVIDIA官方驱动下载链接](https://www.nvidia.com/Download/index.aspx?lang=en-us)检查您的显卡是否适用430及以上版本的 NVIDIA 驱动。</font>

3. 按 Ctrl+Alt+F1 进入命令行界面并关闭图形界面。

   ```bash
   $ sudo service lightdm stop
   ```
   
4. 如果您已经安装 NVIDIA 驱动软件，请删除旧的驱动软件。

   ```bash
   $ sudo apt-get remove nvidia-*
   ```
   
5. 赋予安装文件执行权限并安装驱动软件。下面的示例假设安装文件下载在home目录下。

   ```bash
   $ sudo chmod a+x NVIDIA-Linux-x86_64-430.50.run
   $ sudo ./NVIDIA-Linux-x86_64-430.50.run

6. 重启系统

   ```bash
   $ sudo reboot  
   ```

7. 测试是否安装成功

   ```bash
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

1. 更新源。

   ```bash
   $ sudo apt-get update
   ```

2. 使用 curl 工具下载最新版本的 Docker。

   ```bash
   $ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   Add Docker to your Apt repository.
   $ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   ```

   如果系统中未安装 curl 工具，则先安装 curl, 然后执行上述命令。

   ```bash
   $ sudo apt-get install curl
   ```

3. 更新 apt-get 仓库。

   ```bash
   $ sudo apt-get update
   ```

4. 安装 Docker 及其相关的命令行接口和 runtime 环境。

   ```bash
   $ sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```

5. 重新执行以下命令验证 Docker 是否安装成功。如果能够打印 Docker 的版本信息，则说明已成功安装Docker。

   ```bash
   $ docker -v
   ```

### 安装 NVIDIA container toolkit

1. 使用 curl 添加 gpg key。

   ```bash
   $ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
   sudo apt-key add -
   distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
   ```

2. 更新下载源。

   ```bash
   $ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
   sudo tee /etc/apt/sources.list.d/nvidia-docker.list
   ```

3. 安装 NVIDIA runtime。

   ```bash
   $ sudo apt-get update
   $ sudo apt-get install -y nvidia-container-toolkit
   ```

4. 重启 Docker daemon。

   ```bash
   $ sudo systemctl restart docker
   ```

5. 验证 NVIDIA container toolkit 是否安装成功。

   ```bash
   $ docker run --gpus all nvidia/cuda:9.0-base nvidia-smi
   ```


如果能够成功打印服务器 GPU 信息，则表示安装成功。



## 安装 MegaWise 并导入示例数据

1. 请确保当前用户对目录 `/tmp` 有读写权限。

2. 下载脚本 `install_megawise.sh` 和 `data_import.sh` 至同一目录，并确保当前用户对两个脚本有可执行权限。

   ```bash
   $ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/script/data_import.sh \
   https://raw.githubusercontent.com/Infini-Analytics/infini/master/script/install_megawise.sh
   $ chmod a+x *.sh
   ```
   
3. 安装 MegaWise 并导入示例数据。

   ```bash
   $ ./install_megawise.sh [参数1，必选] [参数2，可选]
   ```

   > 参数1：MegaWise 安装目录的地址，请确保该目录不存在
   
   > 参数2：MegaWise 镜像id，可选，默认'0.4.0'
   
   > 示例：
   
   ```bash
   ./install_megawise.sh  /home/$USER/megawise '0.4.0'
   ```
   
   该语句所执行的操作如下：

   > 1. 拉取 MegaWise docker 镜像。
   > 2. 下载配置文件和示例数据。
   > 3. 启动 MegaWise。
   > 4. 准备示例数据并导入 MegaWise。
   > 5. 修改相关配置参数，重启 MegaWise 服务。

若出现 `Successfully installed MegaWise and imported test data` 则表示 MegaWise 成功安装且示例数据已导入。
