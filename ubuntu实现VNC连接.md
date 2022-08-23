# Linux（ubuntu）

## 开放端口

```
开放 5900 5901
```

## 添加源

```
sudo vim /etc/apt/sources.list
#这个源必须添加，其他仓库没有vncserver
deb http://archive.ubuntu.com/ubuntu/ bionic universe
sudo apt-get update  #更新软件源
```

## 安装VNC4Server

```
apt-get install vnc4server
vncserver   #设置密码
```

```
安装x-windows应用程序
apt-get install -y x-window-system-core
安装GNOME显示管理器gdm3
apt-get install -y gdm3
安装ubuntu桌面
apt-get install -y ubuntu-desktop
安装x-windows应用程序
apt-get install -y gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal
安装GNOME依赖包
apt-get install -y x-window-system-core
```

## 修改配置文件

```
vi ~/.vnc/xstartup

注释掉四行  #网上众说纷纭，搞不清楚，反正注释不注释的没什么明显区别
xsetroot
~
x-windows

---添加
gnome-session &
gnome-panel &
gnome-settings-daemon &
metacity &
nautilus &
gnome-terminal &
```

## 重启以应用

```
关闭已启动的vnc
vncserver -kill :1
启动一个新的vnc
vncserver :1     #端口号为1
```

## VNC客户端登录

```
VNC Servere : ip + port
Name :  everything
```

