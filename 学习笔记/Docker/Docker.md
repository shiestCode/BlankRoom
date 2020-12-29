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

