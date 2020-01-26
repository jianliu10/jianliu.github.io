---
layout: post
title:  "Docker technical notes - docker images"
date:   2020-01-24 00:00:00 -0500
categories: tech-docker-container
---

# docker tool

https://blog.csdn.net/DreamSeeker_1314/article/details/84403166

   
## 存出、载入、上传 image 

如果要导出image (镜像)到本地文件，可以使用docker save命令    
	docker save -o nginx.tar nginx:latest

可以使用docker load将导出的tar文件再导入到local registry     
	docker load < nginx.tar 
	docker load --input nginx.tar

可以使用docker push命令上传镜像到仓库，默认上传到Docker Hub官方仓库(需要登录)。命令格式：
	docker push NAME[:TAG] | [REGISTRY_HOST[:REGISTRY_PORT]/]NAME[:TAG]


##  创建镜像

创建镜像的方法主要有三种:基于已有镜像的容器创建、基于本地模板导入、基于Dockerfile创建（后续详解）。


## 基于已有镜像的容器创建

该方法主要是使用docker commit命令。命令格式为docker commit[OPTIONS]CONTAINER[REPOSITORY[:TAG]]，主要选项包括:

       ·-a，--author="":作者信息;

       ·-c，--change=[]:提交的时候执行 Dockerfile指令，包括CMD| ENTRYPOINT|ENV|EXPOSE|LABEL|ONBUILD|USER|

VOLUME| WORKDIR 等;

        ·-m，--message="":提交消息;

        ·-p，--pause=true:提交时暂停容器运行。


## 基于本地模板导入

        docker  import命令。命令格式为 docker  import [OPTIONS]  file|URL|- [ REPOSITORY [:TAG] ]。要直接导入一个镜像，可以使用OpenVZ提供的模板来创建，或者用其他 已导出的镜像模板来创建。例如：

   cat ubuntu-14.04-x86_64-minimal.tar.gz | docker import - ubuntu:14.04
   

## Dockerfile

8. 使用Dockerfile创建镜像 
      Dockerfile是一个文本格式的配置文件，用户可以使用Dockerfile来快速创建自定义的镜像。

8.1 基本结构
      Dockerfile由一行行命令语句组成，并且支持以#开头的注释行。 一般而言，Dockerfile分为四部分:基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。例如:

   # This Dockerfile uses the ubuntu image
   # VERSION 2 - EDITION 1
   # Author: docker_user
   # Command format: Instruction [arguments / command] ..
   # Base image to use, this must be set as the first line
   FROM ubuntu
   # Maintainer: docker_user <docker_user at email.com> (@docker_user)
   MAINTAINER docker_user docker_user@email.com
   # Commands to update the image
   RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
   RUN apt-get update && apt-get install -y nginx
   RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
   # Commands when creating a new container
   CMD /usr/sbin/nginx
       其中，一开始必须指明所基于的镜像名称，接下来一般是说明维护者信 息。后面则是镜像操作指令，例如RUN指令，RUN指令将对镜像执行跟随的命 令。每运行一条RUN指令，镜像就添加新的一层，并提交。最后是CMD指令，用来指定运行容器时的操作命令。下面是Docker Hub上两个热门镜像的Dockerfile的例子，可以帮助读者对 Dockerfile结构有个基本的认识。

       第一个例子是在debian:jessie基础镜像基础上安装Nginx环境，从而创建 一个新的nginx镜像:

   FROM debian:jessie
   MAINTAINER NGINX Docker Maintainers "docker-maint@nginx.com"
   ENV NGINX_VERSION 1.10.1-1~jessie
   RUN apt-key adv --keyserver hkp://pgp.mit.edu:80 --recv-keys 573BFD6B3D8FBC64107
       9A6ABABF5BD827BD9BF62 \
           && echo "deb http://nginx.org/packages/debian/ jessie nginx" >> /etc/apt/sources.list \
           && apt-get update \
           && apt-get install --no-install-recommends --no-install-suggests -y \
           ca-certificates \
           nginx=${NGINX_VERSION} \
           nginx-module-xslt \
           nginx-module-geoip \
           nginx-module-image-filter \
           nginx-module-perl \
           nginx-module-njs \
           gettext-base \
           && rm -rf /var/lib/apt/lists/*
   # forward request and error logs to docker log collector
   RUN ln -sf /dev/stdout /var/log/nginx/access.log \
       && ln -sf /dev/stderr /var/log/nginx/error.log
   EXPOSE 80 443
   CMD ["nginx", "-g", "daemon off;"]
       第二个例子是基于buildpack-deps:jessie-scm基础镜像，安装Golang相关环境，制作一个GO语言的运行环境镜像:

   FROM buildpack-deps:jessie-scm
   # gcc for cgo
   RUN apt-get update && apt-get install -y --no-install-recommends \
       g++ \
       gcc \
       libc6-dev \
       make \
       && rm -rf /var/lib/apt/lists/*
   ENV GOLANG_VERSION 1.6.3
   ENV GOLANG_DOWNLOAD_URL https://golang.org/dl/go$GOLANG_VERSION.linux-amd64.tar.gz
   ENV GOLANG_DOWNLOAD_SHA256 cdde5e08530c0579255d6153b08fdb3b8e47caabbe717bc7bcd
       7561275a87aeb
   RUN curl -fsSL "$GOLANG_DOWNLOAD_URL" -o golang.tar.gz \
       && echo "$GOLANG_DOWNLOAD_SHA256  golang.tar.gz" | sha256sum -c - \
       && tar -C /usr/local -xzf golang.tar.gz \
       && rm golang.tar.gz
   ENV GOPATH /go
   ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH
   RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"
   WORKDIR $GOPATH
   COPY go-wrapper /usr/local/bin/
8.2 基本指令
1.FROM

      指定所创建镜像的基础镜像，如果本地不存在，则默认会去Docker Hub 下载指定镜像。格式为

      FROM <image>，或FROM <image>:<tag>，或FROM <image>@<digest>。

     任何Dockerfile中的第一条指令必须为FROM指令。并且，如果在同一个Dockerfile中创建多个镜像，可以使用多个FROM指令(每个镜像一次)。

2.MAINTAINER

       指定维护者信息，格式为MAINTAINER<name>。例如:

   MAINTAINER image_creator@docker.com
       该信息会写入生成镜像的Author属性域中。

3.RUN

       运行指定命令。 格式为RUN <command>或 RUN["executable"，"param1"，"param2"]。

      注意，后一个指令会被解析为Json数组，因此必须用双引号。 前者默认将在shell终端中运行命令，即/ bin/sh-c;后者则使用exec执行，不会启动shell环境。 指定使用其他终端类型可以通过第二种方式实现，例如RUN["/bin/bash"，"-c"，"echo hello"]。 每条RUN指令将在当前镜像的基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 \ 来换行。例如:

   RUN apt-get update \
           && apt-get install -y libsnappy-dev zlib1g-dev libbz2-dev \
           && rm -rf /var/cache/apt
4.CMD

    CMD指令用来指定启动容器时默认执行的命令。它支持三种格式:

·CMD["executable"，"param1"，"param2"]使用 exec执行，是推荐使用的方式;
·CMD command param1 param2在/bin/sh中执行，提供给需要交互的应用;
·CMD["param1"，"param2"]提供给 ENTRYPOINT的默认参数。
   每个Dockerfile只能有一条CMD命令。如果指定了多条命令，只有最后一 条会被执行。如果用户启动容器时手动指定了运行的命令(作为run的参数)，则会覆盖掉CMD指定的命令。

5.LABEL

        LABEL指令用来指定生成镜像的元数据标签信息。 格式为LABEL<key>=<value><key>=<value><key>=<value>... 

   LABEL version="1.0"
   LABEL description="This text illustrates \ that label-values can span multiple lines."
6.EXPOSE

      声明镜像内服务所监听的端口。 格式为EXPOSE<port>[<port>...]。 例如:  

   EXPOSE 22 80 8443
   注意，该指令只是起到声明作用，并不会自动完成端口映射。在启动容器时需要使用-P，Docker主机会自动分配一个宿主机的临时端 口转发到指定的端口;使用-p，则可以具体指定哪个宿主机的本地端口会映射过来。
7.ENV

     指定环境变量，在镜像生成过程中会被后续RUN指令使用，在镜像启动 的容器中也会存在。格式为ENV <key> <value>或ENV <key>=<value>... 例如:

   ENV PG_MAJOR 9.3
   ENV PG_VERSION 9.3.4
   RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/
       postgress && ...
   ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
      指令指定的环境变量在运行时可以被覆盖掉，如docker run  --env <key>=<value> built_image。

8.ADD

       该命令将复制指定的<src>路径下的内容到容器中的<dest>路径下。 格式为ADD  <src>  <dest>

       其中<src>可以是Dockerfile所在目录的一个相对路径(文件或目录)，也 可以是一个URL，还可以是一个tar文件(如果为tar文件，会自动解压到<dest> 路径下)。<dest>可以是镜像内的绝对路径，或者相对于工作目录 (WORKDIR)的相对路径。 路径支持正则格式，例如:

   ADD *.c /code/
9.COPY

      格式为COPY<src><dest>。 复制本地主机的<src>(为Dockerfile所在目录的相对路径、文件或目录)下的内容到镜像中的<dest>下。目标路径不存在时，会自动创建。路径同样支持正则格式。当使用本地目录为源目录时，推荐使用COPY。

10.ENTRYPOINT

      指定镜像的默认入口命令，该入口命令会在启动容器时作为根命令执 行，所有传入值作为该命令的参数。  支持两种格式:

ENTRYPOINT ["executable", "param1", "param2"](exec调用执行);
ENTRYPOINT command param1 param2(shell中执行)。
     此时，CMD指令指定值将作为根命令的参数。 每个Dockerfile中只能有一个ENTRYPOINT，当指定多个时，只有最后一个有效。在运行时，可以被-- entrypoint参数覆盖掉，如docker run--entrypoint。

11.VOLUME

    创建一个数据卷挂载点。格式为VOLUME ["/data"]。 可以从本地主机或其他容器挂载数据卷，一般用来存放数据库和需要保存的数据等。

12.USER

       指定运行容器时的用户名或UID，后续的RUN等指令也会使用指定的用户身份。格式为USER daemon。 当服务不需要管理员权限时，可以通过该命令指定运行用户，并且可以在之前创建所需要的用户。例如:

   RUN groupadd -r postgres && useradd -r -g postgres postgres
      要临时获取管理员权限可以使用gosu或sudo。

13.WORKDIR

        为后续的RUN、CMD和 指令配置工作目录。

        ENTRYPOINT 格式为WORKDIR/path/to/workdir。可以使用多个WORKDIR指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如:

   WORKDIR /a
   WORKDIR b
   WORKDIR c
   RUN pwd
则最终路径为 /a/b/c。

14.ARG

      指定一些镜像内使用的参数(例如版本号信息等)，这些参数在执行 docker build命令时才以-- build-arg<varname>=<value>格式传入。格式为ARG<name>[=<default value>]。则可以用docker build--build-arg<name>=<value>.来指定参数值。

15.ONBUILD

       配置当所创建的镜像作为其他镜像的基础镜像时，所执行的创建操作指令。格式为ONBUILD[INSTRUCTION]。 例如，Dockerfile使用如下的内容创建了镜像 image-A:

   [...]
   ONBUILD ADD . /app/src
   ONBUILD RUN /usr/local/bin/python-build --dir /app/src
   [...]
       如果基于image-A创建新的镜像时，新的Dockerfile中使用FROM image-A 指定基础镜像，会自动执行ONBUILD指令的内容，等价于在后面添加了两条 指令:

   FROM image-A
   #Automatically run the following
   ADD . /app/src
   RUN /usr/local/bin/python-build --dir /app/src
       使用ONBUILD指令的镜像，推荐在标签中注明，例如ruby:1.9- onbuild。

16.STOPSIGNAL

       指定所创建镜像启动的容器接收退出的信号值。例如: STOPSIGNAL signal

17.HEALTHCHECK

       配置所启动容器如何进行健康检查(如何判断健康与否)，自Docker 1.12开始支持。格式有两种:

HEALTHCHECK[OPTIONS]CMD command:根据所执行命令返回值是否 为0来判断;
HEALTHCHECK NONE:禁止基础镜像中的健康检查。
     OPTION支持: ·--interval=DURATION(默认为:30s):过多久检查一次; ·--timeout=DURATION(默认为:30s):每次检查等待结果的超时; ·--retries=N(默认为:3):如果失败了，重试几次才最终确定失败。

18.SHELL

     指定其他命令使用shell时的默认shell类型。 默认值为["/bin/sh"，"-c"]。

注意: 对于Windows系统，建议在Dockerfile开头添加#escape=`来指定转义信 息。

 8.3 创建镜像
       编写完成Dockerfile之后，可以通过docker build命令来创建镜像。

       基本的格式为docker build[选项]内容路径，该命令将读取指定路径下 (包括子目录)的Dockerfile，并将该路径下的所有内容发送给 Docker服务端，由服务端来创建镜像。因此除非生成镜像需要，否则一般建议放置 Dockerfile的目录为空目录。有两点经验:

如果使用非内容路径下的Dockerfile，可以通过 -f 选项来指定其路径。
要指定生成镜像的标签信息，可以使用 -t 选项。
    例如，指定Dockerfile所在路径为/ tmp/docker_builder/，并且希望生成 镜像标签为build_repo/first_image，可以使用下面的命令:

   $ docker build -t build_repo/first_image /tmp/docker_builder/
    使用.dockerignore文件 可以通过.dockerignore文件(每一行添加一条匹配模式)来让 Docker忽略匹配模式路径下的目录和文件。例如:

   # comment
       */temp*
       */*/temp*
       tmp?



##
建议读者在生成镜像过程 中，尝试从如下角度进行思考，完善所生成的镜像。

精简镜像用途:  尽量让每个镜像的用途都比较集中、单一，避免构造大而复杂、多功能的镜像;
选用合适的基础镜像:  过大的基础镜像会造成生成臃肿的镜像，一般推 荐较为小巧的debian镜像;
提供足够清晰的命令注释和维护者信息:  Dockerfile也是一种代码，需要 考虑方便后续扩展和他人使用;
正确使用版本号:  使用明确的版本号信息，如1.0，2.0，而非latest， 将避免内容不一致可能引发的惨案;
减少镜像层数:  如果希望所生成镜像的层数尽量少，则要尽量合并指 令，例如多个RUN指令可以合并为一条;
及时删除临时文件和缓存文件:  特别是在执行apt-get指令后，/var/cache/ apt下面会缓存一些安装包;
提高生成􏰀度:  如合理使用缓存，减少内容目录下的文件，或使 用.dockerignore文件指定等;
调整合理的指令顺序:  在开启缓存的情况下，内容不变的指令尽量放在 前面，这样可以尽量复用;
减少外部源的干扰:  如果确实要从外部引入数据，需要指定持久的地 址，并带有版本信息，让他人可以重复而不出错。
