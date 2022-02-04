## 留意你的构建上下文(build命令会将整个上下文的文件发给daemon)
```
docker build . # 以当前目录为上下文
```
* docker build不是运行在docker cli，而是运行在docker daemon;   
* build执行的第一件事是将上下文以递归的方式发送给docker daemon;   
* 通常最好的方式是把dockerfile和必须的文件放在一个空目录，以这个目录为上下文。

## 每一条指令都是独立的
* 注意每一条指令都是独立运行的，所以“RUN cd /tmp”来切换目录对后面指令是没有任何作用的。（切换目录可以考虑用WORKDIR）

## RUN
```shell
RUN command param1 param2 # shell形式，指令是运行在shell里面，默认是/bin/sh -c
RUN ["executable", "param1", "param2"] # exec形式
```
* exec形式没有shell的字符串替换，而且可以在一些没有shell的很基础镜像中执行.  
* exec形式不运行在shell，所以shell的一些程序就没执行，比如RUN [ "echo", "$HOME" ]，就不会替换$HOME变量

## CMD
```shell
CMD ["executable","param1","param2"] # exec形式, 首选模式
CMD ["param1","param2"] # 作为ENTRYPOINT的默认参数
CMD command param1 param2 # shell形式
```
* Dockerfile只能存在一条CMD指令，如果有多条的话只有最后一条指令生效；   
* CMD的主要作用是为容器提供默认的执行命令。这些默认的执行命令可以包含可执行命令，也可以不包含，不过不包含的时候，你必须制定一个ENTRYPOINT指令;  
* 不要混淆RUN和CMD。RUN本质上是在构建阶段运行一个命令并且commit运行结果; CMD则不会在运行时执行任何命令，而是提供给容器一个预期的命令;   

## ENTRYPOINT
```shell
ENTRYPOINT ["executable", "param1", "param2"] # exec形式
ENTRYPOINT command param1 param2 # shell形式
```
* docker run &lt;image&gt;的命令行参数会传给ENTRYPOINT并覆盖所有的CMD参数;比如docker run &lt;image&gt; -d这里的-d就会传给ENTRYPOINT并覆盖CMD
* 如果要修改ENTRYPOINT可以用docker run --entrypoint(只能是exec形式)
* shell形式会阻止任何CMD命令和run(这里指docker run)的命令行参数；但是缺点是shell形式会作为/bin/sh -c子命令来启动，这意味着可执行文件不再是容器的PID 1进程，也就是不能接收到Unix的信号，所以你的可执行程序在docker stop &lt;container&gt;的时候就不会收到SIGTERM，意味着不能让程序“优雅关闭”

## CMD & ENTRYPOINT
* Dockerfile 应该至少指定一个CMD或ENTRYPOINT命令。
* ENTRYPOINT应在将容器用作可执行文件时定义。
* CMD应该用作为ENTRYPOINT命令定义默认参数或在容器中执行临时命令的一种方式。
* docker run的命令行参数会替代CMD

||No ENTRYPOINT|	ENTRYPOINT exec_entry p1_entry|	ENTRYPOINT [“exec_entry”, “p1_entry”]|
|  ----  | ----  | ----  | ----  |
|No CMD	|error, not allowed|	/bin/sh -c exec_entry p1_entry|	exec_entry p1_entry|
|CMD [“exec_cmd”, “p1_cmd”]|	exec_cmd p1_cmd|	/bin/sh -c exec_entry p1_entry|	exec_entry p1_entry exec_cmd p1_cmd|
|CMD [“p1_cmd”, “p2_cmd”]	|p1_cmd p2_cmd	|/bin/sh -c exec_entry p1_entry|	exec_entry p1_entry p1_cmd p2_cmd|
|CMD exec_cmd p1_cmd|	/bin/sh -c exec_cmd p1_cmd	|/bin/sh -c exec_entry p1_entry|	exec_entry p1_entry /bin/sh -c exec_cmd p1_cmd|

## EXPOSE
```shell
EXPOSE <port> [<port>/<protocol>...]
```
* EXPOSE指令并没有真的发布port, 他起的是一个“文档”的作用：作为创建容器的作者“期望”使用者发布的端口
* 要在运行容器时实际发布端口，请使用docker run -p来发布和映射一个或多个端口，或者使用-P标志来发布所有暴露的端口并将它们映射到高阶端口。
## ENV 
```shell
ENV <key>=<value> ...
```  
* ENV设置的环境变量可以在容器被运行的时候访问到，所以如果只是构建时使用的环境变量，请使用ARG，用ARG设置的环境变量在容器运行的时候是不存在的。
## ADD & COPY
```shell
COPY dist/ /app/dist # 将上下文中的dist文件夹内所有内容，按目录结构，拷贝到/app/dist目录下。千万不要写成COPY dist/* /app/dist
```
* ADD可以src一个互联网地址，也可以解压常见的压缩文件
* ADD可以有缓存的功能

