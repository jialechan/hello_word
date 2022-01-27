## 留意你的构建上下文(build命令会将整个上下文的文件发给daemon)
```
# 下面的.就是以当前目录为上下文
docker build .
```
> docker build不是运行在docker cli，而是运行在docker daemon。build执行的第一件事是将上下文以递归的方式发送给docker daemon。通常最好的方式是把dockerfile和必须的文件放在一个空目录，以这个目录为上下文。

## 每一条指令都是独立的
> 注意每一条指令都是独立运行的，所以“RUN cd /tmp”来切换目录对后面指令是没有任何作用的。（切换目录可以考虑用WORKDIR）

## RUN
* RUN <command> (shell模式，指令是运行在shell里面，默认是/bin/sh -c)
* RUN ["executable", "param1", "param2"] (exec模式).
> exec模式没有shell的字符串替换，而且可以在一些没有shell的很基础镜像中执行
> exec模式不运行在shell，所以shell的一些程序就没执行，比如RUN [ "echo", "$HOME" ]，就不会替换$HOME变量

## CMD
* CMD ["executable","param1","param2"] (exec form, this is the preferred form)
* CMD ["param1","param2"] (as default parameters to ENTRYPOINT)
* CMD command param1 param2 (shell form)

Dockerfile只能存在一条CMD指令，如果有多条的话只有最后一条指令生效

The main purpose of a CMD is to provide defaults for an executing container. These defaults can include an executable, or they can omit the executable, in which case you must specify an ENTRYPOINT instruction as well.

Do not confuse RUN with CMD. RUN actually runs a command and commits the result; CMD does not execute anything at build time, but specifies the intended command for the image.
