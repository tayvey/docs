# Linux部署Docker环境数据库

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

