# 制作一个能够在GeneDock平台运行的Docker镜像
本文将详细说明如何在Docker镜像中安装GATK Best Practice需要的软件。

## 安装Docker
如何安装Docker请参考[Docker docs](https://docs.docker.com/)。

## 准备编译上下文
Docker编译的上下文可以是一个本地目录，也可以是一个Git仓库的地址。一般先用一个空目录作为编译上下文，然后把编译需要的文件和Dockerfile添加到这个目录内。我们在本地新建一个目录作为上下文。

```sh
$ pwd
/Users/lim
$ mkdir gatk
$ cd gatk
```
### 准备软件安装包
分析基因组数据的GATK Best Practice流程[用到的软件](https://www.broadinstitute.org/gatk/guide/article?id=2899)有bwa，Picard，SAMtools，R和GATK。
虽然这些包和源代码都可以在容器中通过`wget`和`git`下载，不过我们需要在容器中先安装这些工具。基于让镜像更简单的原则，我们最好把这些东西下载到本地再用`COPY`命令拷贝到容器中。
下载bwa，SAMtools等的源代码到`bioapps`目录。

```sh
gatk
├── Dockerfile
├── bioapps
│   ├── GenomeAnalysisTK.jar
│   ├── bwa # bwa源代码
│   ├── htslib # htslib源代码
│   ├── picard-tools-1.141
│   │   ├── htsjdk-1.141.jar
│   │   ├── libIntelDeflater.so
│   │   ├── picard-lib.jar
│   │   └── picard.jar
│   └── samtools # SAMtools源代码
├── load.R
└── sources.list
```
### 编写Dockerfile
首先指定基础镜像和镜像的作者。

```
FROM ubuntu:14.04
MAINTAINER GeneDock
```
由于Docker官方提供的ubuntu:14.04镜像设置的时区不是东八区，这使得记录日志的时间不是北京时间，所以要把时间改成本地时间。

```
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
另外，官方镜像使用的APT软件源在国内访问会很慢，我们最好换成国内的。大家可以根据自己的网络环境选择一个适合自己的[软件源](http://wiki.ubuntu.org.cn/%E6%BA%90%E5%88%97%E8%A1%A8)。

更新软件源，安装依赖。

```
COPY ./sources.list /etc/apt/sources.list
# 编译SAMtools需要用到Git。安装了Git以后，也可以直接用git从GitHub克隆
# bwa, SAMtools和hitlib的源代码
RUN apt-get update && apt-get install -y gcc \
  make \
  zlib1g-dev \
  libncurses-dev \
  openjdk-7-jre \
  git
```
安装bwa。

```
# 编译时会自动在容器中创建/bioapps/bwa目录
COPY ./bioapps/bwa /bioapps/bwa/
WORKDIR /bioapps/bwa
RUN make -j 8
ENV PATH $PATH:/bioapps/bwa
```
安装SAMtools，注意编译SAMtools时需要htslib的源代码。

```
COPY ./bioapps/htslib /htslib/
COPY ./bioapps/samtools /samtools/
WORKDIR /samtools
RUN make -j 8 && make install
WORKDIR /
```
安装Picard和GATK。

```
COPY ./bioapps/picard-tools-1.141 /bioapps/picard/
COPY ./bioapps/GenomeAnalysisTK.jar /bioapps/gatk/GenomeAnalysisTK.jar
```
安装R和GATK需要的R包。

```
RUN echo "deb http://cran.rstudio.com/bin/linux/ubuntu trusty/" >> /etc/apt/sources.list
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9
RUN apt-get update && apt-get install -y r-base
COPY ./load.R /
RUN Rscript load.R
```
清理安装包。

```
RUN rm -rf /samtoos /htslib load.R && apt-get clean
```
**注意，APP运行前，GeneDock系统会把APP需要的输入文件下载到容器的`/var/data`目录，APP也是在这个目录下执行，所以我们要把容器启动后的所在目录设置成`/var/data`。**

```
WORKDIR /var/data/
```

## 编译镜像
在上下文的根目录(Dockerfile所在的目录)下执行编译命令。

*注意，如果本地没有ubuntu:14.04镜像，编译时首先会从Docker Hub下载这个镜像。由于Docker Hub没有在国内部署服务器或使用国内的CDN服务，在国内下载镜像会非常耗时，不过我们可以选择第三方的加速服务。推荐使用[DaoCloud](https://www.daocloud.io/)或者[阿里百川](http://tae.taobao.com/)的免费加速服务。*

```sh
$ docker build -t genedock/gatk .
Sending build context to Docker daemon  75.1 MB
Sending build context to Docker daemon
Step 0 : FROM ubuntu:14.04
Pulling repository ubuntu
ca4d7b1b9a51: Download complete
2332d8973c93: Download complete
ea358092da77: Download complete
a467a7c6794f: Download complete
Status: Downloaded newer image for ubuntu:14.04
 ---> ca4d7b1b9a51
Step 1 : MAINTAINER GeneDock
 ---> Running in 6b2f2c5a9481
 ---> 66e0e1603968
Removing intermediate container 6b2f2c5a9481
...
```
编译完成以后，启动一个容器验证需要的软件是否都正确安装了。

```sh
$ docker images
REPOSITORY     TAG      IMAGE ID       CREATED          VIRTUAL SIZE
genedock/gatk  latest   c7c52e55b851   17 hours ago     998 MB

$ docker run -it genedock/gatk
root@8a78342c323c:/var/data# bwa

Program: bwa (alignment via Burrows-Wheeler transformation)
Version: 0.7.12-r1044
...

root@8a78342c323c:/var/data# samtools --version
samtools 1.2-232-g87cdc4a
Using htslib 1.2.1-250-gfa6ed9a
Copyright (C) 2015 Genome Research Ltd.

root@8a78342c323c:/var/data# R --version
R version 3.2.2 (2015-08-14) -- "Fire Safety"
Copyright (C) 2015 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu (64-bit)
...

root@8a78342c323c:/var/data# java -jar /bioapps/picard/picard.jar -h
USAGE: PicardCommandLine <program name> [-h]
...

root@8a78342c323c:/var/data# java -jar /bioapps/gatk/GenomeAnalysisTK.jar --version
3.4-46-gbc02625
```

## 上传镜像
目前还不支持从外网上传镜像到GeneDock私有仓库，如果需要上传镜像，请联系我们。

## 附录
### Dockerfile
```
FROM ubuntu:14.04
MAINTAINER GeneDock

RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
COPY ./sources.list /etc/apt/sources.list

# 安装依赖
# 编译SAMtools需要用到Git。安装了Git以后，也可以直接用Git从GitHub克隆
# bwa, SAMtools和hitlib的源代码
RUN apt-get update && apt-get install -y gcc \
  make \
  zlib1g-dev \
  libncurses-dev \
  openjdk-7-jre \
  git

# 安装bwa
COPY ./bioapps/bwa /bioapps/bwa/
WORKDIR /bioapps/bwa
RUN make -j 8
ENV PATH $PATH:/bioapps/bwa

# 安装samtools
COPY ./bioapps/htslib /htslib/
COPY ./bioapps/samtools /samtools/
WORKDIR /samtools
RUN make -j 8 && make install
WORKDIR /

# 安装Picard和GATK
COPY ./bioapps/picard-tools-1.141 /bioapps/picard/
COPY ./bioapps/GenomeAnalysisTK.jar /bioapps/gatk/GenomeAnalysisTK.jar

# 安装R和GATK需要的R包
RUN echo "deb http://cran.rstudio.com/bin/linux/ubuntu trusty/" >> /etc/apt/sources.list
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9
RUN apt-get update && apt-get install -y r-base
COPY ./load.R /
RUN Rscript load.R

RUN rm -rf /samtoos /htslib load.R && apt-get clean
WORKDIR /var/data/
```

### load.R
```
options(repos="http://cran.rstudio.com/")
install.packages("gplots")
install.packages("ggplot2")
install.packages("reshape")
install.packages("gsalib")
```
