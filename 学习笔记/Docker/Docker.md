## 1.安装

------

#### 1.1删除

如果以前安装过就删掉

````bash
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
````

#### 1.2使用存储库安装

在新主机上首次安装Docker Engine之前，需要设置Docker存储库。之后，您可以从存储库安装和更新Docker。

安装`yum-utils`软件包（提供`yum-config-manager` 实用程序）并设置**稳定的**存储库。

````bash
$ sudo yum install -y yum-utils

$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

````

验证一下

````bash
$ docker version
# 或者
$ docker info
````

#### 1.3启动docker服务

````bash
service docekr start
````



````bash
#拉个nginx镜像
docker pull nginx
````

docker 查看日志

````shell
$ docker logs [OPTIONS] CONTAINER
  Options:
        --details        显示更多的信息
    -f, --follow         跟踪实时日志
        --since string   显示自某个timestamp之后的日志，或相对时间，如42m（即42分钟）
        --tail string    从日志末尾显示多少行日志， 默认是all
    -t, --timestamps     显示时间戳
        --until string   显示自某个timestamp之前的日志，或相对时间，如42m（即42分钟）
        
##查看指定时间后的日志，只显示最后100行：        
$ docker logs -f -t --since="2018-02-08" --tail=100 CONTAINER_ID  
##查看最近30分钟的日志:
$ docker logs --since 30m CONTAINER_ID
##查看某时间之后的日志：
$ docker logs -t --since="2018-02-08T13:23:37" CONTAINER_ID
##查看某时间段日志：
$ docker logs -t --since="2018-02-08T13:23:37" --until "2018-02-09T12:23:37" CONTAINER_ID
````

