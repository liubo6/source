---
title: Docker_03_Dockerfile
date: 2016-08-11 10:47:38
categories: 容器
tags: docker
---
手动创建镜像时会用到，创建、提交、搜索、pull和push等命令.除此之外,Docker能自动创建镜像
Docker为我们提供了Dockerfile来解决自动化的问题

Dockerfile包含创建镜像所需要的全部指令。基于在Dockerfile中的指令，我们可以使用`Docker build`命令来创建镜像。通过减少镜像和容器的创建过程来简化部署
<!--more-->

## 语法命令
不区分大小写,命名约定为全部大写
```
INSTRUCTION argument
```

## FROM
所有Dockerfile都必须以FROM命令开始。 FROM命令会指定镜像基于哪个基础镜像创建，接下来的命令也会基于这个基础镜像
FROM命令可以多次使用，表示会创建多个镜像
语法
```
FROM <image name>
```
例如,新的镜像将基于Ubuntu的镜像来构建
```
FROM ubuntu
```

## MAINTAINER
设置该镜像的作者
语法
```
MAINTAINER <author name>
```

## RUN
在shell或者exec的环境下执行的命令。RUN指令会在新创建的镜像上添加新的层面，接下来提交的结果用在Dockerfile的下一条指令中
语法
```
RUN 《command》
```
## ADD
复制文件指令。它有两个参数<source>和<destination>。destination是容器内的路径。source可以是URL或者是启动配置上下文中的一个文件
```
ADD 《src》 《destination》
```

## CMD
提供了容器默认的执行命令。 Dockerfile只允许使用一次CMD指令。 使用多个CMD会抵消之前所有的指令，只有最后一个指令生效
三种形式
```
CMD ["executable","param1","param2"]
CMD ["param1","param2"]
CMD command param1 param2
```
## EXPOSE
指定容器在运行时监听的端口
语法
```
EXPOSE <port>;
```

## ENTRYPOINT
配置给容器一个可执行的命令，这意味着在每次使用镜像创建容器时一个特定的应用程序可以被设置为默认程序。同时也意味着该镜像每次被调用时仅能运行指定的应用。类似于CMD，Docker只允许一个ENTRYPOINT，多个ENTRYPOINT会抵消之前所有的指令，只执行最后的ENTRYPOINT指令
语法
```
ENTRYPOINT ["executable", "param1","param2"]
ENTRYPOINT command param1 param2
```

## WORKDIR
指定RUN、CMD与ENTRYPOINT命令的工作目录
语法
```
WORKDIR /path/to/workdir
```

## ENV
设置环境变量。它们使用键值对，增加运行程序的灵活性
```
ENV <key> <value>
```
## USER
镜像正在运行时设置一个UID
```
USER <uid>
```

## VOLUME
授权访问从容器内到主机上的目录
语法
```
VOLUME ["/data"]
```

保持常见的指令像MAINTAINER以及从上至下更新Dockerfile命令
当构建镜像时使用可理解的标签，以便更好地管理镜像
避免在Dockerfile中映射公有端口
`CMD`与`ENTRYPOINT`命令请使用数组语法

## 示例 java
```
FROM ubuntu:16.04
MAINTAINER liubo <917551811@qq.com>

ENV DEBIAN_FRONTEND noninteractive

ENV VERSION 8
ENV UPDATE 102
ENV BUILD 14

ENV JAVA_HOME /usr/lib/jvm/java-${VERSION}-oracle
ENV JRE_HOME ${JAVA_HOME}/jre

ENV OPENSSL_VERSION 1.0.2h

RUN apt-get update && apt-get install ca-certificates curl \
        gcc libc6-dev libssl-dev make \
        -y --no-install-recommends && \
    curl --silent --location --retry 3 --cacert /etc/ssl/certs/GeoTrust_Global_CA.pem \
    --header "Cookie: oraclelicense=accept-securebackup-cookie;" \
    http://download.oracle.com/otn-pub/java/jdk/"${VERSION}"u"${UPDATE}"-b"${BUILD}"/server-jre-"${VERSION}"u"${UPDATE}"-linux-x64.tar.gz \
    | tar xz -C /tmp && \
    mkdir -p /usr/lib/jvm && mv /tmp/jdk1.${VERSION}.0_${UPDATE} "${JAVA_HOME}" && \
    curl --silent --location --retry 3 --cacert /etc/ssl/certs/GlobalSign_Root_CA.pem \
        https://www.openssl.org/source/openssl-"${OPENSSL_VERSION}".tar.gz \
        | tar xz -C /tmp && \
        cd /tmp/openssl-"${OPENSSL_VERSION}" && \
                ./config --prefix=/usr && \
                make clean && make && make install && \
        apt-get remove --purge --auto-remove -y \
                gcc \
                libc6-dev \
                libssl-dev \
                make && \
    apt-get autoclean && apt-get --purge -y autoremove && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN update-alternatives --install "/usr/bin/java" "java" "${JRE_HOME}/bin/java" 1 && \
    update-alternatives --install "/usr/bin/javac" "javac" "${JAVA_HOME}/bin/javac" 1 && \
    update-alternatives --set java "${JRE_HOME}/bin/java" && \
    update-alternatives --set javac "${JAVA_HOME}/bin/javac"
```

## 示例 tomcat容器
```
FROM liubo6/java-oracle:server_jre_8
MAINTAINER liubo <917551811@qq.com>
ENV CATALINA_HOME /usr/local/tomcat
ENV PATH $CATALINA_HOME/bin:$PATH
RUN mkdir -p "$CATALINA_HOME"
WORKDIR $CATALINA_HOME
 
# see https://www.apache.org/dist/tomcat/tomcat-8/KEYS
RUN set -ex \
    && for key in \
        05AB33110949707C93A279E3D3EFE6B686867BA6 \
        07E48665A34DCAFAE522E5E6266191C37C037D42 \
        47309207D818FFD8DCD3F83F1931D684307A10A5 \
        541FBE7D8F78B25E055DDEE13C370389288584E7 \
        61B832AC2F1C5A90F0F9B00A1C506407564C17A3 \
        79F7026C690BAA50B92CD8B66A3AD3F4F22C4FED \
        9BA44C2621385CB966EBA586F72C284D731FABEE \
        A27677289986DB50844682F8ACB77FC2E86E29AC \
        A9C5DF4D22E99998D9875A5110C01C5A2F6059E7 \
        DCFD35E0BF8CA7344752DE8B6FB21E8933C60243 \
        F3A04C595DB5B6A5F1ECA43E3B7BBB100D811BBE \
        F7DA48BB64BCB84ECBA7EE6935CD23C10D498E23 \
    ; do \
        gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
    done
 
ENV TOMCAT_MAJOR 8
ENV TOMCAT_VERSION 8.0.36
ENV TOMCAT_TGZ_URL https://www.apache.org/dist/tomcat/tomcat-$TOMCAT_MAJOR/v$TOMCAT_VERSION/bin/apache-tomcat-$TOMCAT_VERSION.tar.gz
 
RUN set -x \
    && curl -fSL "$TOMCAT_TGZ_URL" -o tomcat.tar.gz \
    && curl -fSL "$TOMCAT_TGZ_URL.asc" -o tomcat.tar.gz.asc \
    && gpg --batch --verify tomcat.tar.gz.asc tomcat.tar.gz \
    && tar -xvf tomcat.tar.gz --strip-components=1 \
    && rm bin/*.bat \
    && rm tomcat.tar.gz*
 
EXPOSE 8080
CMD ["catalina.sh", "run"]
```
