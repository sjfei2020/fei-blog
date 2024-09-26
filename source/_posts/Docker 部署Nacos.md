---
title: docker部署Nacos
date: 2024-09-26
categories:
  - linux运维


#updated: 2023-07-12
---

# Docker 拉取 Nacos 镜像

在本教程中，我们将以 Nacos 2.1.1 版本为例，介绍如何拉取并启动 Nacos 镜像。

## 拉取 Nacos 镜像

首先，我们可以使用以下命令拉取 Nacos 镜像：

```bash
docker pull nacos/nacos-server:v2.1.1
```

如果您不加版本号，Docker 将默认拉取最新版本的镜像：

```
docker pull nacos/nacos-server
```

## 挂载目录

为了管理 Nacos 的日志和配置文件，我们需要先创建一个目录并给予：

```
mkdir -p /home/nacos/logs/   
```

## 启动 Nacos 并复制文件到宿主机

### 独立模式 - 主机网络

接下来，我们可以启动 Nacos，并将其配置文件和日志复制到宿主机。以下是启动 Nacos 的命令：

```
docker run --name nacos --network host -e MODE=standalone -d nacos/nacos-server:v2.1.1
```

在这个命令中：

- `--name nacos`：指定容器名称。
- `--network host`：使用主机网络。
- `-e MODE=standalone`：设置为独立模式。
- `-d`：以后台模式运行容器。

### 复制配置和日志

启动容器后，我们需要将容器内的配置信息和日志复制到宿主机的 `/home/nacos` 目录下：

```
bash复制代码docker cp nacos:/home/nacos/logs/ /home/nacos/
docker cp nacos:/home/nacos/conf/ /home/nacos/
```

### 移除容器

完成复制后，可以移除容器：

```
docker rm -f nacos
```

## 再次启动 Nacos

为了保证容器能够正常运行，我们需要执行以下命令来再次启动 Nacos：

```
docker run -d --name nacos --network host --privileged=true -e JVM_XMS=256m -e JVM_XMX=256m -e MODE=standalone -v /home/nacos/logs/:/home/nacos/logs -v /home/nacos/conf/:/home/nacos/conf/ --restart=always nacos/nacos-server:v2.1.1
```

在这个命令中：

- `--privileged=true`：扩大容器权限，将容器权限变为 root，避免出现权限不足的问题。
- `-e JVM_XMS=256m` 和 `-e JVM_XMX=256m`：指定 JVM 启动时和运行时的内存分配。
- `-v` 选项用于挂载宿主机的日志和配置目录。



可以查看日志检查Nacos是否正常启动

```
docker logs naocs
```

![image-20240926144926869](C:\Users\IT_FEI\AppData\Roaming\Typora\typora-user-images\image-20240926144926869.png)

## 注意事项

在使用 Nacos 之前，请确保相关端口在防火墙中已开放。如果您使用的是云服务器，还需要配置安全组。以下是开放 Nacos 所需端口的相关命令：

（我的是用的公司分配的超融合服务器，所以直接是映射的主机网络。）

### 开放端口

```
bash复制代码# 开放端口8848 9848 9849
firewall-cmd --zone=public --add-port=8848/tcp --permanent
firewall-cmd --zone=public --add-port=9848/tcp --permanent
firewall-cmd --zone=public --add-port=9849/tcp --permanent
```

### 重启防火墙

```
firewall-cmd --reload
```

### 查看所有开启的端口

```
firewall-cmd --zone=public --list-ports
```

### 重启 Docker

重启完防火墙后，执行以下命令重启 Docker：

```
systemctl restart docker
```

## 访问 Nacos

最后，您可以通过以下链接访问 Nacos：

```
http://localhost:8848/nacos
```

![image-20240926144959611](C:\Users\IT_FEI\AppData\Roaming\Typora\typora-user-images\image-20240926144959611.png)
