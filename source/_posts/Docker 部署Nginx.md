---
title: docker部署Nginx
date: 2024-09-24
categories:
  - linux
#updated: 2023-07-12
---

# 1、运行容器

> docker run --name nginx -d nginx


# 2、创建目录

> mkdir /opt/sevnceWorkplace/docker/nginx

>mkdir /opt/sevnceWorkplace/docker/nginx/html

>mkdir /opt/sevnceWorkplace/docker/nginx/logs

# 3、从容器中复制要挂载的文件
> docker cp nginx:/etc/nginx/conf.d/ /opt/sevnceWorkplace/docker/nginx/

> docker cp nginx:/etc/nginx/nginx.conf /opt/sevnceWorkplace/docker/nginx/

>docker cp nginx:/usr/share/nginx/html/ /opt/sevnceWorkplace/docker/nginx/

>docker cp nginx:/var/log/nginx/ /opt/sevnceWorkplace/docker/nginx/logs/

# 4、停止容器
> docker stop nginx

# 5、删除容器
> docker rm nginx

# 6、重新运行容器

```powershell
docker run --name=nginx \
           -p 80:80 \
           -p 443:443 \
           -v /opt/sevnceWorkplace/frontend:/opt/sevnceWorkplace/frontend \
           -v /opt/sevnceWorkplace/docker/nginx/conf.d:/etc/nginx/conf.d \
           -v /opt/sevnceWorkplace/docker/nginx/nginx.conf:/etc/nginx/nginx.conf \
           -v /opt/sevnceWorkplace/docker/nginx/html:/usr/share/nginx/html \
           -v /opt/sevnceWorkplace/docker/nginx/logs:/var/log/nginx \
           -v /opt/sevnceWorkplace/docker/nginx/cert:/etc/nginx/cert \
           -e TZ=Asia/Shanghai \
           --privileged=true \
           --restart=always \
           -d \
           nginx:1.23.4
```

 - -name：给你启动的容器起个名字，以后可以使用这个名字启动或者停止容器。
- -p：映射端口，将docker宿主机的80端口和容器的80端口进行绑定。
- -v：挂载文件用的。
- 第1个-v挂载配置目录。
- 第2个-v挂载配置文件。
- 第3个-v挂载日志文件。
- 第4个-v挂载页面文件。
- -e TZ=Asia/Shanghai：设置时区。
- -d：后台运行。
- --restart=always：随docker启动。
- --privileged=true：容器内的root拥有真正的root权限。
# 7、测试配置文件格式是否正确
> docker exec nginx nginx -t
# 8、平滑重启Nginx
> docker exec nginx nginx -s reload

