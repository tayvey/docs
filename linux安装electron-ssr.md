# linux安装electron-ssr

## 安装依赖

### arch

```sh
pacman -S fuse nss gdk-pixbuf2 gtk3 alsa-lib python
```

## 授权

```sh
chmod +x electron-ssr-0.3.0-alpha.6.AppImage
```

## systemd配置

```ini
[Unit]

[Service]
Type=simple
User=root
Environment="DISPLAY=:0"
ExecStart=/root/app/ssr/electron-ssr-0.3.0-alpha.6.AppImage --no-sandbox

[Install]
WantedBy=multi-user.target
```