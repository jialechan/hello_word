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
## 将Docker用作轻量级虚拟机
### 可以像一台“普通”的宿主机那样创建一个Docker容器
```shell
# 提取一个正在运行的虚拟机的文件系统
cd / && sudo tar cf /img.tar --exclude=/img.tar --one-file-system /
```
```Dockerfile
# Dockerfile
FROM scratch
ADD img.tar /
```
一些人认为这是一个糟糕的做法，但是它也许可以给你的业务带来收益，或者符合你的用例场景。迁移到Docker的起步阶段时采用将虚拟机转换成Docker镜像的方式相对简单。

### 可以在容器里托管一组服务，运行一个从一开始就运行着多个进程的类宿主机镜像
```shell
# 使用Supervisor在Docker容器里管理多个进程
# 展示如何置备一个同时运行Tomcat和Apache Web服务器的容器，并使用Supervisor应用来管理进程的启动，以托管的方式启动和运行它。
# Dockerfile
FROM ubuntu: 14.04
ENV DEBIAN_FRONTEND noninteractive # 设置一个环境变量，表明此会话是非交互式的
RUN apt-get update && apt-get install -y python-pip apache 2 tomcat7
RUN pip install supervisor
RUN mkdir -p /var/lock/apache2
RUN mkdir -p /var/run/apache2
RUN mkdir -p /var/log/tomcat # 创建一些运行应用所需的维护目录
RUN echo_supervisord_conf > /etc/supervisord.conf # 利用echo_supervisord_conf工具创建一个默认的supervisord配置文件
ADD ./supervisord_add.conf /tmp/supervisord_add.conf # 将Apache和Tomcat的supervisord配置设定复制至镜像里，做好加到默认配置的准备
RUN cat /tmp/supervisord_add.conf >> /etc/supervisord.conf # 将Apache和Tomcat的supervisord配置设定追加到supervisord的配置文件里
RUN rm /tmp/supervisord_add.conf # 由于不再有用处了，删除之前上传的文件
CMD ["supervisord","-c","/etc/supervisord.conf"] 

# supervisord_add.conf
[supervisord] # 为supervisord声明全局酉己置块
nodaemon=true # 设置成不要后台运行Supervisor进程，因为对容器来说它是一个前台进程
# 声明新程序的代码块apache
[program:apache2]
command=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2 -DFOREGROUND" # 用于启动在该代码块中声明的程序的命令
# 声明新程序的代码块tomcat
[program: tomcat] 
command=service start tomcat # 用于启动在该代码块中声明的程序的命令
redirect_stderr=true
stdout_logfile=/var/log/tomcat/supervisor.log
stderr_logfile=/var/log/tomcat/supervisor.error_log # 配置相关日志
```

### 提交是一种正确的随手保存工作的方式。可以使用镜像的构建ID指定从特定的Docker镜像构建。可以给镜像命名。


