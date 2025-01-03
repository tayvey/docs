# WINDOWS使用WSL2

## 命令

```sh
# 更新
wsl --update

# 查看发行版
wsl --list -o

# 查看已安装
wsl --list -v

# 安装 (--web-download 可加可不加, 看网络情况)
wsl --install <发行版名称> [--web-download]

# 导出
wsl --export <已安装发行版名称> "<导出文件路径及名称>.tar"

# 导入
wsl --import <自定义名称> "<执行目录>" "<导入文件路径及名称>.tar"
```

```ini
# 全局配置文件路径: "C:\Users\<当前用户>\.wslconfig"
[wsl2]
# mirrored表示镜像网络, 虚拟机与主机共用IP
networkingMode=mirrored
# 设置wsl的内存大小, 默认为主机内存的一半
memory=16G

[experimental]
# networkingMode=mirrored时此配置才有效. true表示虚拟机和主机之前可以互相通过IP地址访问, 否则只能通过localhost,127.0.0.1访问
hostAddressLoopback=true
```

