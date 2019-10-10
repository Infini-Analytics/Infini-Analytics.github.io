
# 安装 Infini SQL 引擎（ MegaWise ）

本文档将对 MegaWise Docker 容器的安装和配置操作进行介绍，完成本文档中的所有操作后即可连接 MegaWise 进行各类数据操作。文档中涉及的操作主要分为以下部分：

## 安装前提

请确认已安装以下软件包：

- [NVIDIA driver 430](https://github.com/NVIDIA/nvidia-docker/wiki/Frequently-Asked-Questions#how-do-i-install-the-nvidia-driver)

- [Docker 19.03 or higher](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)

- [NVIDIA Container Toolkit](https://github.com/NVIDIA/nvidia-docker)



## MegaWise安装并启动

### 获取MegaWise Docker镜像

1. 确认后台已经运行 Docker daemon

   ```bash
   $ docker version
   ```

   如果没有看到相关服务，请启动 Docker daemon.

2. 获取 MegaWise 镜像 

   ```bash
   $ docker pull zilliz/megawise:0.3.0-d091919-1679
   ```

### 设置MegaWise目录

```bash
# Create MegaWise file
$ mkdir /home/$USER/megawise
$ cd /home/$USER/megawise
$ mkdir conf
```

进入`conf` 目录，下载六个配置文件：

```bash
$ cd conf
$ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/chewie_main.yaml \
https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/etcd.yaml \
https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/megawise_config.yaml \
https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/megawise_config_template.yaml \
https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/render_engine.yaml \
https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/scheduler_config_template.yaml
```

### 设置 MegaWise 启动参数

1. 打开 `conf` 目录下的 `chewie_main.yaml` 配置文件，修改如下片段：

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

   > 注意：`cpu` 部分 `physical_memory` 和 `partition_memory` 应小于服务器的物理内存大小；
   >
   > `gpu_num` 应小于服务器上的显卡数量，`physical_memory` 应小于服务器中全局内存最小的显卡的全局内存大小。

2. 打开 `conf` 目录下面的 `megawise_config.yaml` 配置文件，修改如下片段：

   ```
   worker_num : 1
   gpu:
       physical_memory: 2    # unit: GB
       partition_memory: 2   # unit: GB
   cuda_profile_query_cnt: -1 #-1 means don't profile, positive integer means the number of querys to profile, other value invalid
   ```
   
   > 注意：修改 `worker_num` 、`gpu physical_memory` 和 `partition_memory` 等参数，使其与前面 `chewie_main.yaml` 文件中相应的设置保持一致，其中 `worker_num` 与 `gpu_num` 对应。

### 启动MegaWise

```bash
$ docker run --gpus all --shm-size 17179869184 \ 
-v /home/$USER/megawise/conf:/megawise/conf \ 
-v /home/$USER/megawise/data:/megawise/data \ 
-v /home/$USER/megawise/server_data:/megawise/server_data \ 
-v /tmp:/tmp \ 
-p 5433:5432 \ 
zilliz/megawise:0.3.0-d091919-1679
```

**参数说明**

- `--shm-size`

  Docker image 运行时系统分配的内存大小，可以设置为不超过宿主机可用内存，示例中设置为 16 GB 。

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



## Megawise安装验证

### Docker容器访问

1. 查看MegaWise Docker容器，获取CONTAINER ID信息

   ```bash
   $ docker ps 
   CONTAINER ID	IMAGE	COMMAND		CREATED		STATUS		PORTS	 NAMES
   4aed62f7f5f1 mega1679:0.1.1 "/docker-entrypoint.…" 1 hours ago Up 1 hours 0.0.0.0:5433->5432/tcp herschel
   ```

2. 进入MegaWise Docker容器

   ```bash
   $ docker exec -u megawise -it <CONTAINER ID> bash
   megawise@4aed62f7f5f1:/megawise$
   ```

3. 连接数据库

   ```bash
   megawise@4aed62f7f5f1:/megawise$ cd script
   megawise@4aed62f7f5f1:/megawise/script$ ./connect.sh 
   psql (11.1)
   Type "help" for help.
   postgres=# 
   ```

4. 执行简单sql语句进行测试

   ```sql
   --创建一个名为 aaa 的表
   postgres=# create table aaa(i int);
   CREATE TABLE
   --向该表中插入少量数据
   postgres=# copy aaa from stdin;      
     Enter data to be copied followed by a newline.
     End with a backslash and a period on a line by itself, or an EOF signal.
     >> 1
     >> 2
     >> 3
     >> \.
     COPY 3
   --执行一个简单的查询
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

### 外部访问

1. 确保安装 psql 工具

   ```bash
   $ which psql
   ```

2. 修改MegaWise相关参数

   在宿主机的 `/home/$USER/megawise/data` 目录中，打开 `postgresql.conf` 文件，在末尾加上如下一行：

   ```
   listen_addresses = '*'
   ```

   再打开 `pg_hba.conf` 文件，在末尾加上如下一行：

   ```
   host    all             all             0.0.0.0/0         password
   ```

3. 重启MegaWise

   ```bash
   $ docker restart <CONTAINER ID>
   ```

4. 容器外部访问MegaWise

   ```bash
   $ psql --username=megawise -h 192.168.1.1 -p 5433 -d postgres
   ```

   > 注意: 请把 `192.168.1.1` 改成当前运行MegaWise docker 的服务器的 ip 地址


