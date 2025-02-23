# linux安装electron-ssr

运行文件下载路径<br/>
https://www.123684.com/s/Al49jv-MB2Zv<br/>
提取码:dURa

## 授权

```sh
sudo chmod +x electron-ssr-0.3.0-alpha.6.AppImage
```

## 安装依赖

### arch

```sh
sudo pacman -S fuse nss gdk-pixbuf2 gtk3 alsa-lib python
```

## x11问题处理

```sh
rm -r /tmp/.X11-unix
ln -s /mnt/wslg/.X11-unix /tmp/.X11-unix
```