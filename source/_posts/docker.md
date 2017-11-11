---
title: docker
date: 2017-11-11 20:57:35
tags:
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