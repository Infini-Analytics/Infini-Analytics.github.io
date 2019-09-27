
# 1. 安装 Infini SQL 引擎（ MegaWise ）

## 1.1 环境准备  

- 本文中所涉及的系统，仅限于 Ubuntu 16.04 及以上的版本
- 本文假定你已经在宿主机上安装 nvidia 的驱动，驱动版本为 430 （使用 `nvidia-smi` 命令检查）
- 本文假定你已经在宿主机安装 cuda 工具包，cuda 版本为 9.0 （使用 `cat /usr/local/cuda/version.txt` 命令检查)
- 本文假定你已经在宿主机安装 docker 管理工具，版本为 19.03 （使用 `docker -v` 命令检查）



## 1.2 获取 MegaWise 镜像和配置 docker 环境

### 1.2.1 获取 MegaWise 的镜像

```bash
$ docker pull zilliz/megawise:0.3.0-d091919-1679
```

### 1.2.2 配置 docker 环境

配置 docker 环境仅需要**执行一次**，根据使用的操作系统不同，参考 https://github.com/NVIDIA/nvidia-docker ，下面介绍 Ubuntu 下如何操作：

- 在终端逐条执行以下语句，**需要有 sudo 权限**  

```bash
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)   
$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -   
$ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list  
  
$ sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit  
$ sudo systemctl restart docker   
```
- 检查是否安装成功

```bash
# Test nvidia-smi with the latest official CUDA image  
$ docker run --gpus all nvidia/cuda:9.0-base nvidia-smi  
```
  如果终端中能正确显示显卡相关信息，说明宿主机环境已经准备好了  



## 1.3 初次启动前的准备工作

### 1.3.1 MegaWise 目录设置 

- 我们将创建空白目录 `megawise` ，此目录方便 docker 镜像映射文件，路径为 `/home/zilliz/megawise` ：

```bash
$ mkdir megawise
$ cd megawise
$ mkdir conf
```

+ 进入当前 `megawise` 目录下的 `conf` 目录，获取 megawise 的 6 个配置文件
```bash
$ cd conf
$ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/chewie_main.yaml
$ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/etcd.yaml
$ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/megawise_config.yaml
$ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/megawise_config_template.yaml
$ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/render_engine.yaml
$ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/config/db/scheduler_config_template.yaml
```

### 1.3.2 修改 MegaWise 的启动参数

- 打开 `conf` 目录下面的 `chewie_main.yaml` 配置文件，找到如下片段

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
请根据宿主机的配置情况，设置合理值（数值单位都是 GB ），比如：

`cpu` 部分里 `physical_memory` 和 `partition_memory` 不要设置为超过宿主机当前可用的内存；

`gpu_num` 请不要设置为超过宿主机实际的显卡数量， `physical_memory` 等也不能超过你当前可用的显存  。

- 打开 `conf` 目录下面的 `megawise_config.yaml` 配置文件，找到如下片段

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
修改 `worker_num` 和下面的 `gpu` `physical_memory` 和 `partition_memory` ，与前面 `chewie_main.yaml` 文件中相应的设置保持一致。



## 1.4 初次启动

### 1.4.1 启动容器
+ 使用以下命令启动 MegaWise 的容器

```bash
$ docker run --gpus all --shm-size 17179869184 -v /home/zilliz/megawise/conf:/megawise/conf -v  /home/zilliz/megawise/data:/megawise/data -v /home/zilliz/megawise/server_data:/megawise/server_data -v /tmp:/tmp -p 5433:5432 zilliz/megawise:0.3.0-d091919-1679
```

+ 参数说明
  
    + `--shm-size`
    
      docker image 运行时系统分配的内存大小，可以设置为不超过宿主机可用内存，本教程中设置为 16 GB （ 16\*1024\*1024*1024 ）
    
    + `-v`
    
      宿主机和 image 之间的目录映射，用:隔开，前面是宿主机的目录，后面是 docker image 的目录
    
    + `-p` 
    
      宿主机和 image 之间的端口映射，用:隔开，前面是宿主机的端口，后面是 docker image 的端口，宿主机的端口可以随意设置未被占用的端口
    
    + `-v /tmp:/tmp` 
    
      将 image 中的 `/tmp` 映射到宿主机的 `/tmp` ，这是为了方便查看 MegaWise 的日志，你也可以设置到其他路径下，不过要注意的是你设置的路径需要让当前用户有可读写的权限
    
+ 注意

在启动容器时可以通过 `-v` 将本地存储的数据文件映射到容器内，以实现本地文件导入 MegaWise 数据库。

+ 容器启动后，你会看到满屏的启动日志，找到最关键的那条，这说明 MegaWise server 已经启动成功了

```
Megawise server is running...
```

注意: 请确保本机的防火墙5433端口被打开

### 1.4.2 查看容器
- 运行以下命令查看 docker 容器的运行情况

```bash
  $ docker ps 
  CONTAINER ID IMAGE          COMMAND                CREATED     STATUS     PORTS                  NAMES
  4aed62f7f5f1 mega1679:0.1.1 "/docker-entrypoint.…" 1 hours ago Up 1 hours 0.0.0.0:5433->5432/tcp herschel
```
如果看到如上面的结果说明容器启动成功了

### 1.4.3 进入容器
- 目前容器已经启动，可以进入容器中对 MegaWise 数据库做一些测试， CONTAINER ID 通过 `docker ps` 得到：
```bash
$ docker exec -u megawise -it <CONTAINER ID> bash
  
megawise@4aed62f7f5f1:/megawise$
```
看到上面的结果说明已经进入容器中，并位于 `/megawise/` 目录下，接下来看看进程的启动情况:

```bash
megawise@4aed62f7f5f1:/megawise$ cd script/ && ./ps.sh
  etcd process:
  megawise    17  0.9  0.0 9467448 17048 ?       Sl   09:37   0:12 /megawise/bin/etcd --config-file /megawise/conf/etcd.yaml
  plasma process:
  megawise    60  0.0  0.0 17193108 8024 ?       Sl   09:37   0:00 /megawise/bin/plasma_store_server -m 12884901888 -s /tmp/plasma
  chewie process:
  megawise    61  0.0  0.0 4936156 15128 ?       SLl  09:37   0:00 /megawise/bin/chewie_server --confPath=/megawise/conf/chewie_main.yaml
  postgres process:
  megawise    25  0.0  0.1 434112 46088 ?        S    09:37   0:00 /megawise/bin/postgres -D /megawise/data
  megawise    49  0.0  0.0 434112 10988 ?        Ss   09:37   0:00 postgres: checkpointer   
  megawise    50  0.0  0.0 434244 10988 ?        Ss   09:37   0:00 postgres: background writer   
  megawise    51  0.0  0.0 434112 15904 ?        Ss   09:37   0:00 postgres: walwriter   
  megawise    52  0.0  0.0 434536 13512 ?        Ss   09:37   0:00 postgres: autovacuum launcher   
  megawise    53  0.0  0.0 289052 10408 ?        Ss   09:37   0:00 postgres: stats collector   
  megawise    54  0.0  0.0 434528 12128 ?        Ss   09:37   0:00 postgres: logical replication launcher   
  megawise process:
  root         1  0.0  0.0   4500   780 ?        Ss   09:37   0:00 /bin/sh -c su megawise -c '/megawise/bin/megawise_server -c /megawise/conf/megawise_config.yaml -r /megawise/conf/render_engine.yaml'
  megawise    17  0.9  0.0 9467448 17048 ?       Sl   09:37   0:12 /megawise/bin/etcd --config-file /megawise/conf/etcd.yaml
  megawise    25  0.0  0.1 434112 46088 ?        S    09:37   0:00 /megawise/bin/postgres -D /megawise/data
  megawise    49  0.0  0.0 434112 10988 ?        Ss   09:37   0:00 postgres: checkpointer   
  megawise    50  0.0  0.0 434244 10988 ?        Ss   09:37   0:00 postgres: background writer   
  megawise    51  0.0  0.0 434112 15904 ?        Ss   09:37   0:00 postgres: walwriter   
  megawise    52  0.0  0.0 434536 13512 ?        Ss   09:37   0:00 postgres: autovacuum launcher   
  megawise    53  0.0  0.0 289052 10408 ?        Ss   09:37   0:00 postgres: stats collector   
  megawise    54  0.0  0.0 434528 12128 ?        Ss   09:37   0:00 postgres: logical replication launcher   
  megawise    60  0.0  0.0 17193108 8024 ?       Sl   09:37   0:00 /megawise/bin/plasma_store_server -m 12884901888 -s /tmp/plasma
  megawise    61  0.0  0.0 4936156 15128 ?       SLl  09:37   0:00 /megawise/bin/chewie_server --confPath=/megawise/conf/chewie_main.yaml
  root        63  0.0  0.0  46988  2948 ?        S    09:37   0:00 su megawise -c /megawise/bin/megawise_server -c /megawise/conf/megawise_config.yaml -r /megawise/conf/render_engine.yaml
  megawise    64  0.0  0.2 5502268 68168 ?       Ssl  09:37   0:00 /megawise/bin/megawise_server -c /megawise/conf/megawise_config.yaml -r /megawise/conf/render_engine.yaml
  megawise   231  0.0  0.0  18236  3268 pts/0    Ss   09:57   0:00 bash
  megawise   241  0.0  0.0  18032  2936 pts/0    S+   10:00   0:00 /bin/bash ./ps.sh
  megawise   254  0.0  0.0  34420  2920 pts/0    R+   10:00   0:00 ps aux
```
 从上面可以看到 `etcd/plasma/chewie` 以及 postgres 和 MegaWise 的进程均已经启动了，接下来测试数据库是否能访问，在终端执行如下命令：

  ```bash
  megawise@4aed62f7f5f1:/megawise/script$ ./connect.sh 
  psql (11.1)
  Type "help" for help.
  
  postgres=# 
  ```

  这里使用的是容器中缺省的数据库 postgres ，让我们先创建一个简单的表：

  ```sql
  postgres=# create table aaa(i int);
  CREATE TABLE
  ```
  往里面插入一点数据：

  ```sql
  postgres=# copy aaa from stdin;
  Enter data to be copied followed by a newline.
  End with a backslash and a period on a line by itself, or an EOF signal.
  >> 1
  >> 2
  >> 3
  >> \.
  COPY 3
  ```
  执行一个简单的查询：

  ```sql
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

至此， MegaWise 安装测试成功！

### 1.4.4 停止容器

输入 `exit` 退出容器，在终端停止容器并修改部分设置，然后再次启动

  ```bash
$ docker stop <CONTAINER ID>
  ```



## 1.5 本地访问 docker 中的 MegaWise

### 1.5.1 修改参数
为了让容器中的数据库能从本地访问，需要先修改一些参数，在宿主机的 `megawise/data` 目录中，先打开 `postgresql.conf` 文件

  ```
  # Add settings for extensions here
  shared_preload_libraries = 'zdb_fdw'
  client_min_messages = warning
  enable_mergejoin = off
  enable_nestloop = off
  enable_material = off
  port = 5432
  ```
在文件的末尾加上如下一行

  ```
  listen_addresses = '*'
  ```
再打开 `pg_hba.conf` 文件

  ```
  # Allow replication connections from localhost, by a user with the replication privilege.
  local   replication     all                                     trust
  host    replication     all             127.0.0.1/32            trust
  host    replication     all             ::1/128                 trust
  ```
在文件的末尾加上如下一行（0.0.0.0/0）。这样设置后，可以在本机通过 psql 直接访问容器中的 MegaWise ，而且不用输入密码。如果您需要从其他的机器访问这个容器中的 MegaWise ，请修改这里的 IP 设置。

  ```
  host    all             all             0.0.0.0/0         trust
  ```

### 1.5.2 启动 MegaWise
- 再次启动容器

```bash
$ docker restart <CONTAINER ID>
```

- 本地访问 MegaWise

如果你本机安装了psql（使用 `which psql` 命令检查），就可以在本机通过如下命令访问容器中的数据库了；如果是使用其他的数据库连接工具，数据库的基本配置可以参考下面命令中的参数值

```bash
$ psql --username=megawise -h 192.168.1.65 -p 5433 -d postgres
```

## 1.6 导入测试数据

导入一部分测试数据，以方便之后验证 Infini 图形渲染引擎和前端可视化组件的安装。测试数据抽取自 Uber 开放的纽约出租车订单数据，记录数 100,000 条。

### 1.6.1 新建一个用户
通过以下命令，为 Infini 可视化组件创建一个用户：

```sql
postgres=# CREATE USER zilliz WITH passward 'zilliz';
```

### 1.6.2 创建一个新表

```sql
postgres=# drop table if exists nyc_taxi;
postgres=# create table nyc_taxi(
    vendorID text,
    tpep_pickup_datetime timestamp,
    tpep_dropoff_datetime timestamp,
    passenger_count int,
    trip_distance float,
    
    pickup_longitude float,
    pickup_latitude float,
    dropoff_longitude float,
    dropoff_latitude float,
    fare_amount float,
    tip_amount float,
    total_amount float
    );
```

### 1.6.3 获取测试数据

```bash
$ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/sample_data/nyc_taxi_data.csv
$ docker cp nyc_taxi_data.csv <CONTAINER ID>:/megawise/
```

### 1.6.4 向表中插入测试数据

```sql
postgres=# copy nyc_taxi from '/megawise/nyc_taxi_data.csv WITH DELIMITER ',' csv header;
```
