## 留意你的构建上下文(build命令会将整个上下文的文件发给daemon)
```
# 下面的.就是以当前目录为上下文
docker build .
```
> docker build不是运行在docker cli，而是运行在docker daemon;   
> build执行的第一件事是将上下文以递归的方式发送给docker daemon;   
> 通常最好的方式是把dockerfile和必须的文件放在一个空目录，以这个目录为上下文。

## 每一条指令都是独立的
> 注意每一条指令都是独立运行的，所以“RUN cd /tmp”来切换目录对后面指令是没有任何作用的。（切换目录可以考虑用WORKDIR）

## RUN
* RUN <command> (shell模式，指令是运行在shell里面，默认是/bin/sh -c)
* RUN ["executable", "param1", "param2"] (exec模式).
> exec模式没有shell的字符串替换，而且可以在一些没有shell的很基础镜像中执行.  
> exec模式不运行在shell，所以shell的一些程序就没执行，比如RUN [ "echo", "$HOME" ]，就不会替换$HOME变量

## CMD
* CMD ["executable","param1","param2"] (exec模式, 首选模式)
* CMD ["param1","param2"] (作为ENTRYPOINT的默认参数)
* CMD command param1 param2 (shell模式)

> Dockerfile只能存在一条CMD指令，如果有多条的话只有最后一条指令生效；   
> CMD的主要作用是为容器提供默认的执行命令。这些默认的执行命令可以包含可执行命令，也可以不包含，不过不包含的时候，你必须制定一个ENTRYPOINT指令;  
> 不要混淆RUN和CMD。RUN本质上是在构建阶段运行一个命令并且commit运行结果; CMD则不会在运行时执行任何命令，而是提供给容器一个预期的命令;   

