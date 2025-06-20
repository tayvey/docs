# WINDOWS安装WSL

## WSL常用命令

### 安装WSL

```powershell
wsl --install [发行版本名称]
```

可选参数

|参数|说明|
|-|-|
|--no-distribution|表示仅安装WSL组件, 暂时不安装Linux发行版本<br/>如果不设置, 将默认安装ubuntu|
|--web-download|表示从GitHub安装, 如果不设置将从微软商店安装|

### 更新WSL

```powershell
wsl --update
```

可选参数

|参数|说明|
|-|-|
|--web-download|表示从GitHub安装, 如果不设置将从微软商店安装|

### 查询可安装的发行版本

```powershell
wsl -l -o
```

### 查询已安装的发行版本

```powershell
wsl -l -v
```

### 导出已安装发行版

```powershell
wsl --export <名称> "路径.tar"
```

### 导入

```powershell
wsl --import <自定义名称> "新目录" "路径.tar"
```

## 配置

### 全局配置

`C:\Users\[当前系统用户名]\.wslconfig` 没有就新建一个

```ini
[wsl2]
# mirrored表示镜像网络, 虚拟机与主机共用IP (需要windows11)
networkingMode=mirrored
# 设置Linux的内存大小, 默认为主机内存的一半
memory=32G
# 设置Linux虚拟内存
swap=8GB

[experimental]
# CPU空闲时自动释放缓存. dropcache立即释放
autoMemoryReclaim=dropcache
# networkingMode=mirrored时此配置才有效. true表示虚拟机和主机之前可以互相通过IP地址访问, 否则只能通过localhost,127.0.0.1访问
hostAddressLoopback=true
```

### 独立配置

`/etc/wsl.conf` 没有就新建一个

```ini
[boot]
# 使用systemd (需要wsl版本0.67.6+)
systemd=true

[automount]
# 不将Windows驱动器装载到/mnt下
enabled=false

[network]
# 设置主机名, 默认为Windows主机名
hostname=WSL

[user]
# 设置默认登录用户
default=root

[interop]
# 不添加Windows $PATH
appendWindowsPath=false
```

### 解决图形化程序启动报错

如果启动图形化程序报错"cannot open display: :0", 可以尝试下面的解决办法

修改`/etc/tmpfiles.d/wslg.conf`文件, 如果没有则新建.

在文件中添加以下内容

```conf
L+ /tmp/.X11-unix - - - - /mnt/wslg/.X11-unix
```

## 修改包管理工具镜像源

### Arch

替换文件`/etc/pacman.d/mirrorlist`所有内容

```
Server = http://mirrors.aliyun.com/archlinux/$repo/os/$arch
```

重新同步软件包仓库

```sh
pacman -Syu
```