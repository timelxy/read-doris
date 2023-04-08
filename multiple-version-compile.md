本篇主要记录doris的编译和部署，用于记录各个版本的坑
# Docker CentOS
centos install docker
https://docs.docker.com/engine/install/centos/

use not-root user running docker container
https://docs.docker.com/engine/install/centos/

# Version: apache-doris-1.1.0-bin-x86-jdk8
```
# environment：tencent cloud centos 8.2
# download binary version.  no need to build
...

# run docker container
docker run -it --network host \
     -v /home/${USER}/.m2:/root/.m2 \
     -v /home/${USER}/doris/source/apache-doris-1.1.0-bin-x86-jdk8/:/root/apache-doris-1.1.0-bin-x86-jdk8/ \
     apache/incubator-doris:build-env-ldb-toolchain-latest
```

**遇到问题**
第一次启动时，报ip错误，提示配置priority networks
1. 尝试配了一通，docker container的ip、host ip都试了，CIDR格式，没解决
2. 按照 [docker doc networking提示](https://docs.docker.com/network/network-tutorial-host/) ，启动指定为host ip “--network host”。 解决。   具体原理有待深入研究

# Version: 1.0. Env Tencent Clound VM CentOS8
## Build
```
# download release version from official site
wget https://dlcdn.apache.org/incubator/doris/1.0/1.0.0-incubating/apache-doris-1.0.0-incubating-src.tar.gz

# pull docker image
docker pull apache/incubator-doris:build-env-ldb-toolchain-latest

# run docker container。bind mount .m2 and source code directory
docker run -it -v /home/${USER}/.m2:/root/.m2 -v /your/local/apache-doris-1.0.0-incubating-src/:/root/apache-doris-1.0.0-incubating-src/ apache/incubator-doris:build-env-ldb-toolchain-latest

# compile and package。如果可用内存太小，可能会失败
WITH_MYSQL=1 
sh build.sh
```

## Deploy
按照 [官方文档](https://doris.apache.org/zh-CN/docs/install/install-deploy) 里的步骤，都使用默认配置，单机很快可以部署完成


# Version: 0.8 Env: CentOS
doris官方文档提供的docker env，不能直接build这个0.8.0 beta版本，需要手动做一些事情
```
# env安装Apache Ant
yum install -y ant
```

安装thirdparty依赖

```
# 内容很简单，下载各个依赖，解包。  方便build的时候使用
# 其中有些包下载链接已经失效，或者下载后文件损坏。  遇到了挨个找下新的地址，替换下载
sh thirdparty/build-thirdparty.sh
```

