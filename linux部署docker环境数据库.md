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

