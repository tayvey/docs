# Linux部署Docker环境

## Docker

### 下载二进制包

```sh
wget -p /usr/local/bin https://download.docker.com/linux/static/stable/x86_64/docker-xx.x.x.tgz
```

<span style="color:red">注意替换xx.x.x为实际使用版本号</span>

### 解压并删除压缩包

```sh
tar xvf /usr/local/bin/docker-xx.x.x.tgz --strip-components=1 -C /usr/local/bin/ && rm -rf /usr/local/bin/docker-xx.x.x.tgz
```

<span style="color:red">注意替换xx.x.x为实际使用版本号</span>

### 创建docker配置文件

```sh
mkdir -p /etc/docker && touch /etc/docker/daemon.json
```

### 创建systemd配置文件

```sh
touch /etc/systemd/system/docker.service
```

### 创建docker运行目录 [可选]

```sh
mkdir -p /data/docker
```

### docker配置

```json
{
    "iptables": false,
    "data-root": "/data/docker"
}
```

如果不需要修改docker默认运行目录, 可以省略`data-root`

<span style="color:red">注意配置中不能有注释</span>

### systemd配置

```ini
[Unit]

[Service]
Type=notify
# Environment="HTTP_PROXY=http://your.proxy.server:port"
# Environment="HTTPS_PROXY=https://your.proxy.server:port"
# Environment="NO_PROXY=localhost,127.0.0.1,::1,192.168.1.0/24"
ExecStart=/usr/local/bin/dockerd

[Install]
WantedBy=multi-user.target
```

## Mysql

```sh
# 创建配置文件挂载目录
mkdir -p /data/mysql/conf.d

# 创建数据文件挂载目录
mkdir -p /data/mysql/data

# 创建容器
docker run -d \
--name=mysql \
-p 3306:3306 \
--restart=always \
-e TZ=Asia/Shanghai \
-e MYSQL_ROOT_PASSWORD=123123 \
-v /data/mysql/conf.d:/etc/mysql/conf.d \
-v /data/mysql/data:/var/lib/mysql \
mysql:9.1.0
```

## SqlServer

```sh
# 创建数据挂载目录
mkdir -p /data/mssql/data

# 数据挂载目录授权
chmod -R 777 /data/mssql/data

# 创建容器 (密码需要包含以下四个类别中的至少三个，长度至少为 8 个字符：大写字母、小写字母、数字和非字母数字符号。)
docker run -d \
--name=mssql \
-p 1433:1433 \
--restart=always \
-e TZ=Asia/Shanghai \
-e ACCEPT_EULA=Y \
-e MSSQL_SA_PASSWORD=A123123a \
-v /data/mssql/data:/var/opt/mssql \
mcr.microsoft.com/mssql/server:2022-CU15-GDR1-ubuntu-22.04
```

## Oralce

### 制作镜像

```sh
# 拉取oracle官方镜像制作工具
git clone https://github.com/oracle/docker-images.git

# 去以下地址下载oracle数据库 (不需要解压缩)
# 将压缩包放在镜像制作工具的以下目录 ~/OracleDatabase/SingleInstance/dockerfiles/数据库版本号
https://www.oracle.com/database/technologies/oracle-database-software-downloads.html

# 跳转至 ~/OracleDatabase/SingleInstance/dockerfiles 目录
# 授予制作工具执行权限
chmod +x ./buildContainerImage.sh

# 开始制作
./buildContainerImage.sh -v 数据库版本号 -e
```

### 部署

```sh
# 创建数据挂载目录
mkdir -p /data/oracle/data

# 数据挂载目录授权
chmod -R 777 /data/oracle/data

# 创建容器
docker run -d \
--name=oracle \
-p 1521:1521 \
--restart=always \
-e TZ=Asia/Shanghai \
-e ORACLE_SID=ORCLS \
-e ORACLE_PDB=ORCLP \
-e ORACLE_PWD=123123 \
-v /data/oracle/data:/opt/oracle/oradata \
twyyy/oracle:21c
```

### 远程连接

OCI下载地址: [Oracle Instant Client Downloads](https://www.oracle.com/database/technologies/instant-client/downloads.html)

如果使用的Oracle Instant Client版本为19c及以上, 则有可能会出现"ora-12637 包接收失败"异常.

解决方案如下

```
在oracle服务端的数据库文件目录中找到sqlnet.ora文件
添加以下配置关闭OOB检查, 这是19c及以上版本包含的功能
DISABLE_OOB_AUTO=TRUE
```

配置Oracle Instant Client

```
在 network/admin 目录下新建一个文件 tnsnames.ora , 文件内容如下

连接名称=主机地址:端口号/SID名称

连接名称=
(DESCRIPTION = 
  (ADDRESS = (PROTOCOL = TCP)(HOST = 主机地址)(PORT = 端口号))
  (CONNECT_DATA =
    (SERVER = DEDICATED)
    (SERVICE_NAME = PDB名称)
  )
)
```

配置PL/SQL (每次修改配置需要重启PL/SQL)

```
在PL/SQL的"配置 => 首选项 => Oracle => 连接"中修改以下配置

Oracle主目录=Oracle Instant Client根目录
OCI库=Oracle Instant Client根目录下的oci.dll文件
环境变量
 TNS_ADMIN=tnsnames.ora文件所在目录
 NLS_LANG=SIMPLIFIED CHINESE_CHINA.ZHS16GBK
```

## PostgreSql

```sh
# 创建数据挂载目录
mkdir -p /data/pgsql/data

# 创建容器
docker run -d \
--name=pgsql \
-p 5432:5432 \
--restart=always \
-e TZ=Asia/Shanghai \
-e POSTGRES_PASSWORD=123123 \
-v /data/pgsql/data:/var/lib/postgresql/data \
postgres:17.0
```

## Redis

### 创建配置文件和持久化数据文件挂载

```sh
mkdir -p /root/app/redis/data && touch /root/app/redis/redis.conf
```

### 创建容器

```sh
docker run -d \
--name=redis \
--network=host \
--restart=always \
-e TZ=Asia/Shanghai \
-v /root/app/redis/data:/data \
-v /root/app/redis/redis.conf:/usr/local/etc/redis/redis.conf \
redis \
redis-server /usr/local/etc/redis/redis.conf
```

### 初始化

#### 进入容器

```sh
docker exec -it redis bash
```

#### 连接Redis实例

```sh
redis-cli
```

### 创建用户

```sh
ACL SETUSER root on >123123 +@all ~*
```

## MongoDB

### 创建数据挂载目录

```sh
mkdir -p /root/app/mongo/data
```

### 创建副本集KEY文件

```sh
openssl rand -base64 756 > /root/app/mongo/data/mongo-rs.key && chmod 400 /root/app/mongo/data/mongo-rs.key
```

### 创建docker容器

```sh
docker run -d \
--name=mongo \
--network=host \
--restart=always \
-e TZ=Asia/Shanghai \
-v /root/app/mongo/data:/data/db \
mongo \
--auth \
--replSet rs \
--keyFile /data/db/mongo-rs.key \
--port 27017
```

### 初始化

#### 进入容器

```sh
docker exec -it mongo bash
```

#### 连接数据库实例

```sh
mongosh
```

#### 切换数据库

```sh
use admin
```

#### 初始化副本集

```sh
rs.initiate({_id:"rs", members:[{_id:0, host:"127.0.0.1:27017", priority:2}]})
```

#### 创建用户

```sh
db.createUser({ user:"root",pwd:"123123",roles:[{role:"root",db:"admin"}]})
```

## Nginx

```sh
# 临时运行nginx容器
docker run -d --rm --name=nginx nginx:1.27.2

# 创建nginx挂载目录
mkdir -p /data/nginx

# 复制默认静态文件夹
docker cp nginx:/usr/share/nginx/html /data/nginx/

# 复制默认配置文件
docker cp nginx:/etc/nginx/nginx.conf /data/nginx/

# 复制默认扩展配置文件
docker cp nginx:/etc/nginx/conf.d /data/nginx/

# 创建日志挂载目录
mkdir -p /data/nginx/log

# 创建日志文件
> /data/nginx/log/access.log
> /data/nginx/log/error.log

# 停止临时nginx容器
docker stop nginx

# 创建正式nginx容器
docker run -d \
--name=nginx \
-p 80:80 \
-p 443:443 \
--restart=always \
-e TZ=Asia/Shanghai \
-v /data/nginx/html:/usr/share/nginx/html \
-v /data/nginx/nginx.conf:/etc/nginx/nginx.conf \
-v /data/nginx/conf.d:/etc/nginx/conf.d \
-v /data/nginx/log:/var/log/nginx \
nginx:1.27.2

# 整合
docker run -d --rm --name=nginx nginx:1.27.4 && \
mkdir -p /data/nginx && \
docker cp nginx:/usr/share/nginx/html /data/nginx/ && \
docker cp nginx:/etc/nginx/nginx.conf /data/nginx/ && \
docker cp nginx:/etc/nginx/conf.d /data/nginx/ && \
mkdir -p /data/nginx/log && \
> /data/nginx/log/access.log && \
> /data/nginx/log/error.log && \
docker stop nginx && \
docker run -d \
--name=nginx \
-p 80:80 \
-p 443:443 \
--restart=always \
-e TZ=Asia/Shanghai \
-v /data/nginx/html:/usr/share/nginx/html \
-v /data/nginx/nginx.conf:/etc/nginx/nginx.conf \
-v /data/nginx/conf.d:/etc/nginx/conf.d \
-v /data/nginx/log:/var/log/nginx \
nginx:1.27.4
```

## OceanBase

```sh
# 创建数据挂载目录
mkdir -p /data/oceanbase/data

# 创建配置挂载目录
mkdir -p /data/oceanbase/cluster

# 创建容器
docker run -d \
--name=oceanbase \
-p 2881:2881 \
--restart=always \
--ulimit nofile=20000:20000 \
--ulimit nproc=120000:120000 \
-e TZ=Asia/Shanghai \
-e MODE=NORMAL \
-e EXIT_WHILE_ERROR=false \
-v /data/oceanbase/data:/root/ob \
-v /data/oceanbase/cluster:/root/.obd/cluster \
quay.io/oceanbase/oceanbase-ce
```