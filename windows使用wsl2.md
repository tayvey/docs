# WINDOWS使用WSL2

## wsl常用命令

```powershell
# windows powershell

# 安装wsl2 
# --no-distribution表示仅安装wsl2组件, 暂时不安装linux发行版本, 如果不设置, 将默认安装ubuntu
wsl --install [--no-distribution]

# 更新wsl2组件
wsl --update

# 查看可安装的linux发行版本
wsl -l -o

# 查看已安装的linux发行版本
wsl -l -v

# 安装linux发行版本
# --web-download表示从网络安装, 否则将会从微软商店安装
wsl --install <发行版名称> [--web-download]

# 导出
wsl --export <已安装发行版名称> "<导出文件路径及名称>.tar"

# 导入
wsl --import <自定义名称> "<存放目录>" "<导入文件路径及名称>.tar"
```

## 全局配置
```ini
# 全局配置文件路径: "C:\Users\<当前用户>\.wslconfig"

[wsl2]
# mirrored表示镜像网络, 虚拟机与主机共用IP
networkingMode=mirrored
# 设置wsl的内存大小, 默认为主机内存的一半
memory=32G
# 设置虚拟内存
swap=8GB

[experimental]
# CPU空闲时自动释放缓存. dropcache立即释放
autoMemoryReclaim=dropcache
# networkingMode=mirrored时此配置才有效. true表示虚拟机和主机之前可以互相通过IP地址访问, 否则只能通过localhost,127.0.0.1访问
hostAddressLoopback=true
```

## 安装arch发行版

安装文件下载地址<br/>
https://github.com/yuk7/ArchWSL/releases

```txt
下载`Arch.zip`压缩包并解压, 将会得到`Arch.exe`文件和`rootfs.tar.gz`文件

可以自定义`Arch.exe`文件名称`<自定义>.exe`, 后面创建的arch wsl实例将会使用这个名称

可以将以上两个文件放置在任意目录下, 后面创建的wsl虚拟磁盘文件将会自动创建在执行目录下

运行`Arch.exe`创建wsl实例以及虚拟磁盘文件
```

[可选] 修改windows终端中的配置图标<br/>
https://gitlab.archlinux.org/uploads/-/system/group/avatar/23/iconfinder_archlinux_386451.png?width=48

设置root用户密码

```sh
# wsl arch

passwd
```

[可选] 创建常用用户以避免直接使用root用户

```sh
# wsl arch

# 创建`wheel`用户组并授权
echo "%wheel ALL=(ALL) ALL" > /etc/sudoers.d/wheel

# 添加用户
useradd -m -G wheel -s /bin/bash <用户名>

# 设置用户密码
passwd <用户名>
```

```powershell
# windows powershell

# 设置默认用户. 在`Arch.exe`目录下执行
.\Arch.exe config --default-user <用户名>
```

[可选] 初始化pacman密钥环 (取决于是否需要使用pacman)

```sh
# wsl arch

# 初始化 GPG 密钥环
sudo pacman-key --init

# 导入官方软件仓库的 GPG 公钥
sudo pacman-key --populate

# 同步软件包仓库
sudo pacman -Syy archlinux-keyring

# 设置arch linux阿里巴巴源
# 编辑/etc/pacman.d/mirrorlist，注释掉里面的所有行, 然后在文件的最顶端添加
# Server = http://mirrors.aliyun.com/archlinux/$repo/os/$arch
# ctrl + s 保存, ctrl + x 退出
sudo nano /etc/pacman.d/mirrorlist

# 重新同步软件包仓库
sudo pacman -Syyu
```

解决图形化程序可能会无法使用的问题

```sh
# wsl arch

# 创建文件 /etc/tmpfiles.d/wslg.conf
> /etc/tmpfiles.d/wslg.conf

# 写入内容
echo "L+ /tmp/.X11-unix - - - - /mnt/wslg/.X11-unix" >> /etc/tmpfiles.d/wslg.conf
```

`arch.exe`常用命令

```powershell
# windows powershell

# 备份
.\Arch.exe backup [--tar | --tgz]

# 卸载
.\Arch.exe clean

# 从备份安装
.\Arch.exe install <备份文件>

# 获取guid
.\Arch.exe get --lxguid

# 设置默认用户
.\Arch.exe config --default-user <用户名>
```