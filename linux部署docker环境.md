# Linux部署Docker环境

## Docker

```sh
# 下载安装包 & 解压
cd /usr/local/bin
curl -O https://download.docker.com/linux/static/stable/x86_64/docker-27.3.1.tgz
tar xvf docker-27.3.1.tgz
mv ./docker ./docker.d
mv ./docker.d/* ./
rm -rf ./docker-27.3.1.tgz
rm -rf ./docker.d

# 创建docker配置文件
mkdir -p /etc/docker
> /etc/docker/daemon.json

# 创建systemd配置文件
> /etc/systemd/system/dockerd.service

# 创建docker运行目录
mkdir -p /data/docker
```

```json
// daemon.json
{
    "iptables": false
}
```

```ini
# dockerd.service
[Unit]

[Service]
Type=notify
# Environment="HTTP_PROXY=http://your.proxy.server:port"
# Environment="HTTPS_PROXY=https://your.proxy.server:port"
# Environment="NO_PROXY=localhost,127.0.0.1,::1,192.168.1.0/24"
ExecStart=dockerd --data-root /data/docker

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

```sh
# 创建配置文件挂载目录
mkdir -p /data/redis/conf.d

# 创建配置文件
> /data/redis/conf.d/redis.conf

# 设置密码
echo requirepass 123123 >> /data/redis/conf.d/redis.conf

# 创建持久化数据存储挂载目录
mkdir -p /data/redis/data

# 创建容器
docker run -d \
--name=redis \
-p 6379:6379 \
--restart=always \
-e TZ=Asia/Shanghai \
-v /data/redis/data:/data \
-v /data/redis/conf.d:/usr/local/etc/redis \
redis:7.4.1 \
redis-server /usr/local/etc/redis/redis.conf
```

## MongoDB

```sh
# 创建数据挂载目录
mkdir -p /data/mongo/data

# 创建副本集KEY文件
openssl rand -base64 756 > /data/mongo/data/mongo-rs.key

# 副本集KEY文件授权
chmod 400 /data/mongo/data/mongo-rs.key

# 创建容器
docker run -d \
--name=mongo \
-p 27017:27017 \
--restart=always \
-e TZ=Asia/Shanghai \
-v /data/mongo/data:/data/db \
mongo:8.0.3 \
--auth \
--replSet rs \
--keyFile /data/db/mongo-rs.key

# 进入容器
docker exec -it mongo bash

# 连接mongodb
mongosh
```

```js
// 跳转数据库
use admin

// 初始化副本集
rs.initiate({_id:"rs", members:[{_id:0, host:"127.0.0.1:27017"}]})

// 初始化默认用户
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
```

