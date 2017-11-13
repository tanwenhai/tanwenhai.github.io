---
title: docker
date: 2017-11-11 20:57:35
tags: docker
---
# 容器
### 运行容器
```bash
docker run -itd twh/java 
```
### 查看容器
```bash
docker ps -a
```
### 删除容器
```bash
docker rm $(docker container ls -a -q)
```
### 连接后台运行中的容器
```bash
docker attach twh/java
```
> **该命令有时候不方便，因为是同步的，若有多个用户attach到一个容器，一个窗口命令阻塞，其他窗口都无法执行**

```bash
docker exec -it twh/java
```
### 导出容器
```bash
docker export 容器ID > java.tar.gz
```

# 镜像
### 查找镜像
```bash
docker search mysql -s 100
```
### 拉取镜像
```bash
docker pull mysql
```
### 删除镜像
```bash
docker rmi 镜像ID
```

### 保存镜像
```bash
docker save -o java.tar.gz twh/java
```
### 导入镜像
```bash
cat java.tar.gz|docker import - twh/java:v1.0.1
```
# docker 私有仓库搭建
1. 获取registry镜像
```bash
docker pull registry
```
2. 运行镜像 -p端口映射 -v数据卷
```bash
docker run -d -p 5000:5000 -v /data/docker/registry:/tmp/registry registry
```
3. 上传镜像
```bash
docker push 192.168.3.3:5000/java
```
# 容器互联
1. 要实现容器互联，需要为容器指定一个方便记忆的名字，通过--name来指定
```bash
docker run -d -it --name test1 gitlab/gitlab-ce
```
2. 启动容器
```bash
docker run -itd --name mysql -v /data/docker/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mysql
```
3. 连接容器 --link 容器名:别名
```bash
docker run -d -it --name test1 --link mysql:mysql gitlab/gitlab-ce
```
4. 查看hosts和环境变量
```bash
cat /etc/hosts
```
# Dockerfile
1. 基础镜像信息
```Dockerfile
# 这里注释
# 第一行必须指定基础镜像可以多个
FROM centos 
# 维护者信息
MAINTAINER tanwenhai tanwenhai@outlok.com
# 镜像操作指令
VOLUME /tmp
ADD demo-0.0.1-SNAPSHOT.jar jar
ENV JAVA_OPTS ""
RUN yum -y install mysql
EXPOSE 8080:8080
# 容器启动指令
CMD ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```
2. 语法
* **FROM** 指定基础镜像

    > 格式为FROM image或FROM image:tag

* **MAINTAINER** 维护者信息
    
    > MAINTAINER 用户名 邮箱
* **RUN** 镜像操作

    *run指令在build具有缓存*

    > RUN command 或RUN ["EXECUTABLE", "param1"...] 后者使用exec运行

* **CMD** 启动命令

    *多个CMD命令只执行最后一个*

    > CMD 或RUN ["EXECUTABLE", "param1"...] 使用exec运行    
    CMD command param1 ...      
    CMD ["param1", ...] 提供给ENTERYPOINT的默认参数
* **EXPOSE** 暴露端口
    > EXPOSE port [port/protocol...]
* **ENV** 环境变量
    
    > ENV key value    
    ENV key=value ...
* **ADD** 复制本地目录中的文件到容器中的dest

    > ADD src dest
* **COPY** 和ADD一样 dest不存在是会自动创建目录

    > COPY src dest

* **ENTRYPOINT** 容器启动后执行的命令

    > ENTRYPOINT ["executable", "param1", ...] 
* **volume** 创建数据卷

    > VOLUME ["/data"]
* **USER** 指定容器运行时的用户名或UID

    > USER username
* **WORKDIR** 为后续的RUN、CMD、ENTRYPOINT指定配置工作目录
    
    > WORKDIR /APP
3. 构建镜像 

    *-t 指定奖项的标签信息*

    > docker build -t centos_test .