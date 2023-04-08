本篇主要记录doris开发和持续集成环境搭建。面向源码调试、改bug、测试提交pr等场景

# 1. Mac
## 1.1. Machine
MacBook Pro Intel/M1
MacOS 12.5.1

## 1.2. FE Local
[参考](https://doris.apache.org/zh-CN/community/developer-guide/fe-idea-dev/)

遇到问题
1. M1 Mac，执行 `mvn clean install -DskipTests` ，会有proto相关的几个包找不到arm版本。 maven官方库找到有arm包的新版本，替换POM.xml里的 `dependency version` 可解
2. 文档里提示先执行 `mvn generate-sources`, 失败了再执行 `mvn clean install -DskipTests`。这块不是很理解。  如果要从source code编译完整的包，应该直接执行后者，完整编译。 前者只能生成protobuf、thrift等模板代码。 除非直接下载官网编译好的二进制包，直接运行
3. mvn的JDK环境，需要和最后run的JDK环境一致
最终运行成功

## 1.3. BE
目标：启个BE，供FE连接。这样至少可以在IDEA里单步调试FE。  BE的调试尚未考虑
### 1.3.1. Local
用官方binary包，以及自己local 编译，都比较困难。C++环境依赖严重。卒

### 1.3.2. BE Docker
使用doris官方的1.1.0 image（docker pull apache/doris:build-env-for-1.1.0），直接启动官网的[1.1.0 binary包](https://archive.apache.org/dist/doris/1.1/1.1.0-rc05/apache-doris-1.1.0-bin-x86-noavx2-jdk11.tar.gz)

问题1: 启动BE后，外部IDEA启动的FE无法与BE连接。
解决：https://golangexample.com/connect-directly-to-docker-for-mac-containers-via-ip-address/

问题2: 连接成功，show backends显示backend alive。但是创建表报错，BE的日志里。也显示无法反着向FE 9020发同步状态
暂未解决。大概率还是container和host的网络问题，难搞

# Ubuntu

## machine
tencent 轻量应用服务器
Ubuntu Server 20.04 LTS 64bit

## docker
[参考](https://doris.apache.org/zh-CN/community/developer-guide/docker-dev)

### install docker engine on ubuntu
[参考](https://docs.docker.com/engine/install/ubuntu/)

[using docker as non-root user](https://docs.docker.com/engine/install/linux-postinstall/)

平添太多麻烦，因此暂时直接使用root

### 