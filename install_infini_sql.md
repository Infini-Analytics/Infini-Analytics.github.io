
# 安装 Infini SQL 引擎（ MegaWise ）

本文档将对 MegaWise Docker 容器镜像的安装和配置操作进行介绍，完成本文档中的所有操作后即可连接 MegaWise 进行各类数据操作。文档中涉及的操作主要分为以下部分：

- **安装前提**。该部分用于安装和配置 MegaWise Docker 镜像的运行环境，主要包括了 NVIDIA 驱动、Docker 和 NVIDIA Docker runtime 的安装。
- **MegaWise Docker 镜像的获取和配置**。该部分包括了镜像的获取、Docker 设置、目录设置、MegaWise 启动参数设置四个部分。
- **安装验证**。启动容器并检查容器的运行情况和配置的正确性。

## 安装前提

- 运行 MegaWise Docker 镜像要求服务器的操作系统为 Ubuntu 16.04 及以上版本。
- 禁用 Nouveau 驱动，并安装 NVIDIA driver 430，具体参照 [安装 NVIDIA 驱动](#安装NVIDIA驱动) 。如已安装，请使用 `nvidia-smi` 命令检查是否安装成功。
- [安装 Docker](#安装Docker)，版本不低于1.12，建议安装 Docker 19.03。如已安装，请使用 `docker -v` 命令检查。
- [安装NVIDIA container toolkit](#安装NVIDIAcontainertoolkit)。

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
如果系统中已安装有 Docker 且 Docker 版本不低于1.12，则可跳过 Docker 安装步骤，开始[安装 NVIDIA container toolkit](#安装NVIDIAcontainertoolkit)。如果系统中未安装 Docker，则参照以下步骤开始安装 Docker。

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


## MegaWise Docker 镜像获取和配置  
### 获取 Docker 镜像

1. 通过以下命令从 Docker hub 获取 MegaWise 的镜像 ：

   ```
   docker pull zilliz/megawise:0.3.0-d091919-1679
   ```

2. 获取 MegaWise 镜像的信息。
   通过 Docker images 查看的镜像ID（如下面终端结果中的 `0523f2b43444`），记录镜像的信息，在之后的操作中将会使用到。

   ```
   user@server:~/Downloads$ docker images
   REPOSITORY          TAG                     IMAGE ID            CREATED             SIZE
   zilliz/megawise     0.3.0-d091919-1679      0523f2b43444        5 hours ago         4.95GB
   ```

### 设置目录

在服务器上创建一个空白的目录 ` megawise`，这个目录将作为 MegaWise Docker 镜像时映射的文件和目录的总目录。在本文档的后续示例中，假设该目录的路径为 `/home/data/megawise` 。

```
$ mkdir megawise
$ cd megawise
$ mkdir conf
```

进入当前 `megawise` 目录下的 `conf` 目录，获取 Megawise 的 6 个配置文件：

```
$ cd conf
$ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/chewie_main.yaml
$ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/etcd.yaml
$ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/megawise_config.yaml
$ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/megawise_config_template.yaml
$ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/render_engine.yaml
$ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/scheduler_config_template.yaml
```



在data目录下创建3个空白的子目录，分别是config，data和server_data

### 设置 MegaWise 启动参数

1. 打开 `conf` 目录下面的 `chewie_main.yaml` 配置文件，找到如下片段：

   ```
   cache:  # size in GB
     cpu:
         physical_memory: 16
         partition_memory: 16
   
     gpu:
         gpu_num: 1
         physical_memory: 2
         partition_memory: 2 
   ```

   请根据服务器的硬件配置情况，对上述的配置项进行设置（数值单位都是 GB ），比如：

   `cpu` 部分里 `physical_memory` 和 `partition_memory` 应小于服务器的物理内存大小；`gpu_num` 应小于服务器上的显卡数量，`physical_memory` 应小于服务器中全局内存最小的显卡的全局内存大小。

2. 打开 `conf` 目录下面的 `megawise_config.yaml` 配置文件，找到如下片段：

   ```
   worker_config:
   bitcode_lib: /megawise/lib
   precompile: true
   stage:
       build_task_context_parallelism: 1
       fetch_meta_parallelism: 1
       compile_parallelism: 1
       fetch_data_parallelism: 1
       compute_parallelism: 1
       output_parallelism: 1
   worker_num : 1
   gpu:
       physical_memory: 2    # unit: GB
       partition_memory: 2   # unit: GB
   cuda_profile_query_cnt: -1 #-1 means don't profile, positive integer means the number of querys to profile, other value invalid
   ```

   修改 `worker_num` 、`gpu physical_memory` 和 `partition_memory` 等参数，使其与前面 `chewie_main.yaml` 文件中相应的设置保持一致。其中 `worker_num` 与 `gpu_num` 对应。

### 启动容器
启动 MegaWise 的容器（路径和镜像 ID 使用文档中的示例）

```
docker run --gpus all --shm-size 17179869184 \ 
                        -v /home/data/megawise/conf:/megawise/conf \ 
                        -v /home/data/megawise/data:/megawise/data \ 
                        -v /home/data/megawise/server_data:/megawise/server_data \ 
                        -v /tmp:/tmp \ 
                        -p 5433:5432 \ 
                        0523f2b43444
```

如果 Docker 版本低于19.03，则执行以下命令：

```
  docker run --runtime=nvidia --shm-size 17179869184 \ 
                        -v /home/data/megawise/conf:/megawise/conf \ 
                        -v /home/data/megawise/data:/megawise/data \ 
                        -v /home/data/megawise/server_data:/megawise/server_data \ 
                        -v /tmp:/tmp \ 
                        -p 5433:5432 \ 
                        0523f2b43444
```
**参数说明**

- `--shm-size`

  Docker image 运行时系统分配的内存大小，可以设置为不超过宿主机可用内存，本教程中设置为 16 GB 。

- `-v`

  宿主机和 image 之间的目录映射，用 `:` 隔开，前面是宿主机的目录，后面是 Docker image 的目录。

- `-p`

  宿主机和 image 之间的端口映射，用 `:` 隔开，前面是宿主机的端口，后面是 Docker image 的端口，宿主机的端口可以随意设置未被占用的端口。

- `-v /tmp:/tmp`

  将 image 中的 `/tmp` 映射到宿主机的 `/tmp` ，这是为了方便查看 MegaWise 的日志，你也可以设置到其他路径下，不过要注意的是你设置的路径需要让当前用户有可读写的权限。

> 注意: 在启动容器时可以通过 `-v` 将本地存储的数据文件映射到容器内，以实现本地文件导入 MegaWise 数据库。

容器启动后，将会启动日志，如果能找到如下日志内容，则说明 MegaWise server 已经启动成功。

```
Megawise server is running...
```

### 查看容器
运行以下命令查看 Docker容器的运行情况：

  ```
user@server:~/data/megawise$ docker ps 
CONTAINER ID IMAGE          COMMAND                CREATED     STATUS     PORTS                  NAMES
4aed62f7f5f1 mega1679:0.1.1 "/docker-entrypoint.…" 1 hours ago Up 1 hours 0.0.0.0:5433->5432/tcp herschel
  ```
如果看到类似上面的结果，则说明容器的启动成功了。

### 停止容器

在终端中输入 exit 离开容器环境，通过以下命令停止容器。

```
 user@server:~/data/megawise$ docker stop 4aed62f7f5f1 
```

### 修改 PostgreSQL 的相关参数

MegaWise 使用 PostgreSQL 的前端来接收和解析 SQL 请求。为了让容器中的数据库能从 Docker 容器外部访问，需要修改一些 PostgreSQL 相关的参数。

1. 在宿主机的 `megawise/data` 目录中，先打开 `postgresql.conf` 文件，包含如下内容：

   ```
     # Add settings for extensions here
     shared_preload_libraries = 'zdb_fdw'
     client_min_messages = warning
     enable_mergejoin = off
     enable_nestloop = off
     enable_material = off
     port = 5432
   ```

2. 在文件的末尾加上如下一行。

   ```
     listen_addresses = '*'
   ```

3. 再打开 `pg_hba.conf` 文件。

   ```
   # Allow replication connections from localhost, by a user with the replication privilege.
   local   replication     all                                     trust
   host    replication     all             127.0.0.1/32            trust
   host    replication     all             ::1/128                 trust
   ```

   在文件的末尾加上如下一行。这样设置后，可以从服务器通过 psql 工具直接访问容器中的 MegaWise。

   ```
   host    all             all             0.0.0.0/0         password
   ```

## 安装验证

使用之前提到过的 Docker 命令再次启动容器。容器启动后在终端中执行以下命令进入容器。

  ```
user@server:~/data/megawise$ docker exec -u megawise -it 4aed62f7f5f1 bash
megawise@4aed62f7f5f1:/megawise$
  ```
看到上面的结果说明你已经进入容器中，并位于 `/megawise/` 目录下；进入 `script` 目录，执行如下命令即可连接到容器中缺省的数据库 Postgres。

  ```
  megawise@4aed62f7f5f1:/megawise/script$ ./connect.sh 
  psql (11.1)
  Type "help" for help.
  
  postgres=# 
  ```

  可通过一些简单的命令测试检查 MegaWise 是否正常运行。比如在以下的示例中，先创建一个名为 `aaa` 的表：

  ```
  postgres=# create table aaa(i int);
  CREATE TABLE
  ```
  向该表中插入少量数据：

  ```
  postgres=# copy aaa from stdin;      
  Enter data to be copied followed by a newline.
  End with a backslash and a period on a line by itself, or an EOF signal.
  >> 1
  >> 2
  >> 3
  >> \.
  COPY 3
  ```
  执行一个简单的查询，检查是否能够正确返回结果。

  ```
postgres=# select * from aaa;
INFO:  Number of results set : 3
INFO:  <<<<< GPU Acceleration >>>>>
i 
---
1
2
3
(3 rows)
  ```

如果服务器上安装了 psql 工具，可以在服务器上通过以下命令检查是否可从容器外部访问 MegaWise，其中`192.168.1.65` 为假设的服务器 IP 地址。如果实际服务器的 IP 地址与假设不同，则将其替换为 MegaWise 容器所在的服务器地址即可。`5433` 为服务器上的端口号，请确保其与之前启动容器的操作中与 `MegaWise Docker` 容器的端口进行关联的端口号一致。

  ```
psql --username=megawise -h 192.168.1.65 -p 5433 -d postgres
  ```
