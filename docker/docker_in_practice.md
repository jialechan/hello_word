## 理解Docker -- 深入引擎室
### 你可以向外界开放Docker守护进程API，他们所需要的是以某种方式发起HTTP请求 -- 一个网页浏览器就足够了。
```shell
# 先关闭docker守护进程
sudo service docker stop
# 或
systemctl stop docker

# 对外开放Docker守护进程
sudo docker daemon -H tcp://0.0.0.0:2375
# 连接远端的docker守护进程；
docker -H tcp://<宿主机IP>:2375 <subcommand>
# 如果可以不用sudo运行docker命令，可以设置DOCKER_HOST简化命令
export DOCKER_HOST=tcp://<宿主机的IP>:2375
docker <subcommand>
```

### 容器不是非得接管你的终端，你可以在后台启动它们，并在后面找回。
```shell
# -d将以守护进程方式运行容器
docker run -d
```

### 你可以使用用户自定义网络（推荐方法）让容器进行通信; 或使用链接非常明确地控制容器间通信。
```shell
# 创建用户自定义网络
docker network create my_network
# 将blog1连接到新网络中
docker network connect my_network blog1
# 新起一个容器连接到新网络中, curl blog1即可通讯
docker run -it --network my_network ubuntu:16.04 bash

# 以下是使用链接非常明确地控制容器间通信，非推荐
docker run --name wp-mysql -e MYSQL_ROOT_PASSWORD=mypassword -d mysql
docker run --name workpress --link wp-mysql:mysql -p 10003:80 -d wordpress
```

### 由于Docker守护进程API是基于HTTP的，如果你有任何问题，可以简单地通过一些网络监控工具来调试。
```shell
# socat是一个用于调试和跟踪网络调用的特别有用的工具。
# -v 提高可读性；UNIX-LISTEN让socat在Unix套接字上进行监听；
# fork确保socat不回在首次请求后退出；
# UNIX-CONNECT是让socat连接到Docker的Unix套接字
socat -v UNIX-LISTEN:/tmp/dockerapi.sock,fork UNIX-CONNECT:/var/run/docker.sock &

# 测试连接 
docker -H unix:///tmp/dockerapi.sock ps -a
```
