# Dockerfile参考
Docker可以通过读取`Dockerfile`中的命令自动创建镜像。`Dockerfile`是一个文本文件，它包括用户使用命令行创建镜像时调用的所有命令。使用`docker build`命令，用户可以创建一个连续执行多条命令的自动编译。

## 使用Dockerfile
`docker build`命令从一个`Dockerfile`和上下文编译镜像。编译的上下文通常是一个指定的`PATH`（本地文件系统的一个目录）或`URL`（Git仓库的地址）下的所有文件。上下文会被递归的处理，因此`PATH`会包括任何子目录，`URL`会包括所有的子模块。下面是用当前目录作为上下文进行编译的例子：

```
$ docker build .
Sending build context to Docker daemon  6.51 MB
...
```
Docker守护进程执行编译的过程。编译的第一步是递归的把上下文发送给Docker守护进程。通常，最好在开始时用一个空目录作为上下文，并把Dockerfile放在这个目录下，然后只添加编译镜像需要的文件到这个目录。
> 警告：不要使用`/`目录作为`PATH`，因为这样会把你硬盘上所有的文件发送给Docker守护进程。

为了使用编译上下文中的文件，`Dockerfile`会在命令（比如`COPY`）中特别指明需要的文件。在上下文的目录中添加`.dockerignore`文件可以排除一些文件和目录，这样可以提高编译的性能。如何创建`.dockerignore`文件请参考[https://docs.docker.com/engine/reference/builder/#dockerignore-file][1]。

通常，`Dockerfile`会被命名为`Dockerfile`并放在上下文的根目录。可以在`docker build`命令中使用`-f`指定文件系统中任意的`Dockerfile`。

```
$ docker build -f /path/to/a/Dockerfile .
```

可以使用`-t`给新编译的镜像指定仓库和标签。

```
$ docker build -t genedock/baseimage .
```

Docker守护进程一条条的执行`Dockerfile`中的命令，在最终输出新镜像的ID之前，它会在需要的情况下把每条命令的执行结果提交成一个新的镜像。守护进程会自动清理你发送的上下文。

由于每条命令的执行都是相互独立的并会创建一个新的镜像，所以`RUN cd /tmp`不会对后面的命令造成任何影响。

```
Sending build context to Docker daemon 2.048 kB
Step 0 : FROM ubuntu:14.04
 ---\> ca4d7b1b9a51
Step 1 : MAINTAINER Li Ming, liming@genedock.com
 ---\> Running in 9f854423a6bd
 ---\> 0aede3502f4f
Removing intermediate container 9f854423a6bd
Step 2 : RUN cd /tmp
 ---\> Running in 701e8fb2355b
 ---\> 5f2517db0ccb
Removing intermediate container 701e8fb2355b
Step 3 : RUN echo $PWD
 ---\> Running in a3bb600cbd34
/
 ---\> f28b0ad55ab2
Removing intermediate container a3bb600cbd34
Successfully built f28b0ad55ab2
```

Docker会尽可能的使用中间镜像（缓存）来加速`docker build`过程。控制台打印的`Using cache`信息会指出这个过程。

```
$ docker build -t test .
Sending build context to Docker daemon 2.048 kB
Step 0 : FROM ubuntu:14.04
 ---\> ca4d7b1b9a51
Step 1 : MAINTAINER Li Ming, liming@genedock.com
 ---\> Using cache
 ---\> 0aede3502f4f
Step 2 : RUN cd /tmp
 ---\> Using cache
 ---\> 5f2517db0ccb
Step 3 : RUN echo $PWD
 ---\> Using cache
 ---\> f28b0ad55ab2
Successfully built f28b0ad55ab2
```

## Dockerfile格式
下面是`Dockerfile`的格式：

```
# Comment
INSTRUCTION arguments
```
`INSTRUCTION`不区分大小写，但管理是`INSTRUCTION`全部大写，这样很容易把它们和参数区分开。

Docker顺序的执行`Dockerfile`中的命令。**第一条命令必须是用`FROM`指定基础镜像**。

Docker会把`#`开头的行当作注释，不在一行开头出现的`#`都会被当作参数。这样可以允许下面的声明：

```
# Comment
RUN echo 'we are running some # of cool things'
```
### 环境变量替换
在一些命令中可以把环境变量（通过`ENV`声明）当做`Dockerfile`能够解释的变量使用。对变量类似语法的转义也会被处理。

在`Dockerfile`中，环境变量被标记成`$variable_name`或者`${variable_name}`的形式。这两种形式是等价的，花括号的语言一半用在变量名和后续的字符之间没有空格的情况，例如`${foo}_bar`。

`{$variable_name}`语法同时支持一些标准`bash`的修饰：

 + `${variable:-word}`：如果已经设置了`variable`，结果就是那个变量的值，否则结果会是`word`
 + `${variable:+word}`：如果已经设置了`variable`，结果会是`word`，否则结果是空字符串

在所有情况下，`word`可以是任意的字符串，可以是其他的环境变量。

转义可以通过在变量前加``来实现：`\$foo`或是`${foo}`，分别会被翻译成`$foo`和`${foo}`。例如：

```
FROM genedock/baseimage
ENV foo /bar
WORKDIR ${foo}     # WORKDIR /bar
ADD .$foo          # ADD . /bar
COPY \$foo /quux   # COPY $foo /quux
```

下面的`Dockerfile`命令都支持环境变量：

+ ADD
+ COPY
+ ENV
+ EXPOSE
+ LABEL
+ USER
+ WORKDIR
+ VOLUME
+ STOPSIGNAL
+ ONBUILD (只有和上面的命令联合使用才可以)

在一条命令中，环境变量的值会保持一致。可以看下面的例子：

```
ENV abc=hello
ENV abc=bye def=$abc
ENV ghi=$abc
```
`def`的值会是`hello`而不是`bye`。`ghi`的值会是`bye`，因为它不是把`abc`设置成`bye`那条命令的一部分。

## FROM
```
FROM <image>
# or
FROM <image>:<tag>
# or
FROM <image>@<digest>
```
`FROM`给后续的命令设置基础镜像。因此，一个合法的`Dockerfile`的第一条命令必须是`FROM`。

+ `FROM`必须是`Dockerfile`中第一个非注释的命令
+ `FROM`在一个`Dockerfile`中可以出现多次，以便创建多个镜像。
+ `tag`和`digest`是可选的。如果`tag`和`digest`都没有提供，Docker默认找`tag`为`latest`的镜像。如果找不到匹配的`tag`，Docker会返回错误。

## MAINTAINER
```
MAINTAINER <name>
```
`MAINTAINER`可以让你给生成的镜像设置作者。

## RUN
`RUN`有两种形式：

+ `RUN <command>`: shell形式，<command>会在一个shell（`/bin/sh -c`）中执行
+ `RUN ["executable", "param1", "param2"]`: exec形式

`RUN`命令会在当前镜像上创建一个新层来执行命令并提交执行的结果。新提交的镜像在执行`Dockerfile`的下一步时使用。

exec形式可以避免修改shell字符串，也可以在没有`/bin/sh`的基础镜像中执行shell命令。

在shell形式中，可以使用``把一条`RUN`命令写成多行。例如，下面的两行，

```
RUN /bin/bash -c 'source $HOME/.bashrc ;\\
echo $HOME'
```
和下面的一行是等价的

```
RUN /bin/bash -c 'source $HOME/.bashrc ; echo $HOME'
```
> 注意
>
> + 如果要使用其他的shell，而不是`/bin/sh`，需要使用exec形式传递要使用的shell。例如，`RUN ["/bin/bash", "-c", "echo hello"]`
> + exec形式会以JSON数组的形式处理，所以需要用双引号`"`把单词引起来。
> + 和shell形式不一样的是，exec不会调用shell命令。这意味着常规的shell处理不会发生。比如，`RUN ["echo", "$HOME"]`不会对`$HOME`进行变量替换。如果要进行shell处理，你需要直接使用shell形式或者直接执行shell。比如，`RUN ["sh", "-c", "echo", "$HOME"]`

`RUN`命令的缓存在接下来的编译中不会自动失效。像`RUN apt-get dist-upgrade -y`的缓存在接下来的编译中会被复用。可以使用`--no-cache`标记让`RUN`命令的缓存失效。例如，`docker build --no-cache`。更多信息请查看[`Dockerfile`最佳实践][2]。

## CMD
`CMD`有三种形式：

+ `CMD ["executable", "param1", "param2"]`：exec形式，这是推荐的形式
+ `CMD ["param1", "param2"]`：会作为`ENTRYPOINT`的默认参数
+ `CMD param1 param2`：shell形式

一个`Dockerfile`最多只能有一个`CMD`命令。如果你写了多个`CMD`命令，只有最后一个会被执行。`CMD`的主要目的提供启动容器时执行的命令。如果已经申明了`ENTRYPOINT`，`CMD`提供的命令会被忽略。

> 注意
> + 如果`CMD`用来给`ENTRYPOINT`提供默认参数，`CMD`和`ENTRYPOINT`都需要用JSON数据格式。
> + exec形式会以JSON数组的形式处理，所以需要用双引号`"`把单词引起来。
> + 和shell形式不一样的是，exec不会调用shell命令。这意味着常规的shell处理不会发生。比如，`CMD ["echo", "$HOME"]`不会对`$HOME`进行变量替换。如果要进行shell处理，你需要直接使用shell形式或者直接执行shell。比如，`CMD ["sh", "-c", "echo", "$HOME"]`

如果想让你的容器每次都执行相同的可执行文件，应该考虑结合使用`ENTRYPOINT`和`CMD`。

用户在`docker run`中提供的参数会覆盖`CMD`声明的默认参数。
> `RUN`会实际执行命令并提交结果。在编译的时候，`CMD`不会执行任何东西，它只给镜像指定了打算要执行的命令。

## LABEL
```
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```
`LABEL`用来给镜像添加元数据。一个`LABEL`是一个键值对。一个例子，

```
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \\
that label-values can span multiple lines."
```

## EXPOSE
```
EXPOSE <port> [<port>...]
```
`EXPOSE`命令告诉Docker容器在运行时监听的特定端口号。`EXPOSE`不会让容器的端口在宿主机网络可以访问，需要使用`-p`暴露一部分或者`-P`暴露所有的端口给宿主机网络。如何在宿主机建立重定向端口，请参考[-P标记][3]。

## ENV
```
ENV <key> <value>
ENV <key>=<value> ...
```
`ENV`用来设置环境变量，设置的环境变量可以被后续的命令使用，也会在启动的容器中保存。`ENV`有两种形式，第一种形式`ENV <key> <value>`只能设置一个变量，第二种形式`ENV <key>=<value> ...`可以在一行设置多个变量。

## ADD
`ADD`有两种形式

+ `ADD <src>... <dest>`
+ `ADD ["<src>", ... "<dest>"]`(路径有空格时需要使用这种形式)

`ADD`用来把`<src>`指定的文件、目录或是URL添加到容器文件系统的`<dest>`路径下。如果`<src>`是文件或者目录时，需要使用相对于编译上下文的相对路径。`<drc>`需要是绝对路径或者是相对于`WORKDIR`的相对路径。

`ADD`遵循下面的规则：

+ `<src>`的路径必须在编译上下文内部。由于编译的第一步是把编译上下文发送给docker守护进程，所以不能使用`ADD ../something/something`。
+ 如果`<src>`是URL并且`<dest>`没有以`/`结尾，那么从URL下载的文件会保存成`<dest>`。
+ 如果`<src>`是URL并且`<dest>`以`/`结尾，从UR下载的文件会保存成`<dest>/<filename>`，`<filename>`会从URL中推断。
+ 如果`<src`是目录，目录中所有的内容都会被拷贝，但不包括目录本身。
+ 如果`<src`是本地一个`tar`压缩（`gzip`, `bzip2`或者`xz`）的文件，ADD会把它解压成一个目录。从URL下载的文件不会被解压。
+ 如果`<dest>`没有以`/`结尾，它会当作普通文件，`<src>`的内容会被写到`<dest>`。
+ 如果`<dest>`不存在，docker会创建路径上所有缺失的目录。

## COPY
`COPY`有两种形式

+ `COPY <src>... <dest>`
+ `COPY ["<src>", ... "<dest>"]`(路径有空格时需要使用这种形式)
`<src>`可以是相对于编译上下文的文件或者目录。`COPY`不能用来下载文件，也不会解压`tar`压缩的文件，其他功能都和`ADD`类似。推荐使用`COPY`，因为它的方式更透明。

## ENTRYPOINT
`ENTRYPOINT`有两种形式

+ `ENTRYPOINT ["executable", "param1", "param2"]`：exec形式
+ `ENTRYPOINT command param1 param2`：shell形式
`ENTRYPOINT`用来配置启动容器后执行的命令。只有Dockerfile中最后的`ENTRYPOINT`命令有用。

## VOLUME
```
VOLUME ["/data"]
```
`VOLUME`命令用来创建一个可以从宿主机或其他容器挂载的挂载点。

## USER
```
USER daemon
```
`USER`用来指定运行容器时的用户或者UID，后续的`RUN`，`CMD`和`ENTRYPOINT`也会使用这个用户。

如果服务不需要管理员权限，可以使用`USER`切换成非管理员用户。在使用之前需要先创建用户，例如`RUN groupadd -r postgres && useradd -r -g postgres postgres`。如果需要使用管理员权限，可以使用`gosu`，不推荐使用`sudo`。

## WORKDIR
```
WORKDIR /path/to/workdir
```
`WORKDIR`用来给后续的`RUN`，`CMD`，`ENTRYPOIN`设置工作目录。

## ARG
```
ARG <name>[=\<default value]
```
`ARG`用来定义用户在编译时通过`docker build --build-arg <varname>=<value>`传值的变量。

## ONBUILD
```
ONBUILD [INSTRUCTION]
```
`ONBUILD`用来配置当前镜像作为基础镜像创建其他镜像时需要执行的触发器命令。触发器命令会在下游的编译上下文中执行，就像是它被添加到了下游`Dockerfile`的`FROM`命令之后。

## 参考

1. [Dockerfile reference][4]
2. [Best practices for writing Dockerfiles][5]
3. [Docker——从入门到实践][6]


[1]:	https://docs.docker.com/engine/reference/builder/#dockerignore-file
[2]:	https://docs.docker.com/engine/articles/dockerfile_best-practices/#build-cache
[3]:	https://docs.docker.com/engine/reference/run/#expose-incoming-ports
[4]:	https://docs.docker.com/engine/reference/builder/
[5]:	https://docs.docker.com/engine/articles/dockerfile_best-practices/
[6]:	https://www.gitbook.com/book/yeasy/docker_practice/details
