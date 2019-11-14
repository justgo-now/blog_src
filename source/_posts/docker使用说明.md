---
title: docker使用说明
date: 2019-10-30 21:12:14
updated: 2019-11-1 17:01:54 #手动添加更新时间
top: 1
category: 持续集成
tags: 
- docker
---

## 查看docker运行信息

docker inspect 会返回一个 JSON 文件记录着 Docker 容器的配置和状态信息

```
docker inspect NAMES 
# 查看容器所有状态信息；

docker inspect --format='{{.NetworkSettings.IPAddress}}' ID/NAMES
# 查看 容器ip 地址

docker inspect --format '{{.Name}} {{.State.Running}}' NAMES
# 容器运行状态
```

<!-- more -->

**查看进程信息**

```
docker top NAMES
```

**查看端口；(使用容器ID 或者 容器名称)**

```
docker port ID/NAMES
```

**查看IP地址 也可以直接通过用 远程执行命令也可以**（Centos7）；

```
docker exec -it ID/NAMES ip addr 
```



## docker 的安装及使用

### 简单介绍

> docker 是一个开源的软件部署解决方案
> docker 也是轻量级的应用容器框架
> docker 可以打包、发布、运行任何的应用

### 安装

- 阿里云

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

- daocloud

```
 curl -sSL https://get.daocloud.io/docker | sh
```

安装后将会自动重启

### 卸载

```
sudo apt-get remove docker docker-engine
rm -fr /var/lib/docker/
```

### 配置加速器

下面是我的配置，实际使用需要根据自己的账号去查看自己的地址

- [DaoCloud](https://www.daocloud.io/mirror#accelerator-doc)

```
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://ced808ab.m.daocloud.io
sudo systemctl restart docker.service
```

- [阿里云](https://cr.console.aliyun.com/cn-hangzhou/mirrors)

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://dist7hw1.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 基础命令

- 查看版本:`docker -v` //文章使用版本：Docker version 18.06.0-ce, build 0ffa825

- 查看镜像：`docker images`

- 查看容器：`docker ps`

- 启动 docker 服务：`sudo service docker start`

- 停止 docker 服务：`sudo service docker stop`

- 重启 docker 服务：`sudo service docker restart`

- 进入一个运行中的容器：`docker exec -it 容器Id /bin/bash`

### 通过 Dockerfile 使用 nginx

通过下面的一个脚本可以简单快速的创建一个镜像并运行起来
大概看下应该就可以大概明白镜像的基本使用了

```
echo '0.创建测试目录及代码'
mkdir dockerfiletest
cd dockerfiletest
mkdir dist
echo 'hello world'>./dist/index.html

echo '1.创建Dockerfile'
echo '
From daocloud.io/library/nginx:1.13.0-alpine
COPY dist/ /usr/share/nginx/html/
'>./Dockerfile

echo '2.构建镜像'
docker build -t dockerfiletest .

echo '3.运行镜像'
docker run -p 3344:80 dockerfiletest
```

下面分步拆解下

#### 1.添加 Dockerfile 文件

详细请参考：https://hub.daocloud.io/repos/2b7310fb-1a50-48f2-9586-44622a2d1771

html 的简单部署

```
From daocloud.io/library/nginx:1.13.0-alpine
# 将发布目录的文件拷贝到镜像中
COPY dist/ /usr/share/nginx/html/
```

若要使用自己的配置脚本，比如 vue 的配置,可以将自己的配置文件复制到容器中

```
From daocloud.io/library/nginx:1.13.0-alpine
# 删除镜像中 nginx 的默认配置
RUN rm /etc/nginx/conf.d/default.conf
# 复制 default.conf 到镜像中
ADD default.conf /etc/nginx/conf.d/
# 将发布目录的文件拷贝到镜像中
COPY dist/ /usr/share/nginx/html/
```

nginx 中 vue history 模式的配置 如下，可参考

```
server {
    listen       80;
    location / {
        root /usr/share/nginx/html/;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
}
```

~~若是将`/usr/share/nginx/html/`和`/etc/nginx/conf.d/`挂载到本地，这样应该能够灵活使用 docker 安装的 nginx 了(未实践过)~~

#### 2.构建镜像

构建参数说明参考：http://www.runoob.com/docker/docker-build-command.html

```
docker build -t docker-nginx-test .
```

#### 3.运行镜像

--name 服务名
-d 后台运行
-p 暴露端口:nginx 端口
docker-nginx-test 镜像名/IMAGE ID

```
docker run --name dockertest -d -p 4455:80 docker-nginx-test
```

#### 4.测试访问

```
root@ubuntu:~# curl http://localhost:4455
hello world
```

> 现在，可以通过 IP+端口的形式在外网访问站点了，但在实际使用肯定还需要绑定域名等一些操作
> 最简单的是我认为是使用 nginx 去做代理
> 目前我们公司使用的 [traefik](https://traefik.io/) ，最爽的莫过于 https 的支持，可以了解一下



#### 5.基础操作

1 **docker images** 查看镜像信息列表 镜像是静态的

2 **docker ps -a** 查看运行中的所有容器

3 **docker pull  [images]:[version]**从dockerhub拉取指定镜像

4 **docker run -p 8000:80 -tdi --privileged [imageID] [command]**  后台启动docker,并指定宿主机端口和docker映射端口。

 **-i:**以交互模式运行容器，通常与 -t 同时使用；

 **-d:**后台运行容器，并返回容器ID；

**-t:**为容器重新分配一个伪输入终端，通常与 -i 同时使用；

**--privileged** 容器将拥有访问主机所有设备的权限

通常情况下 [command] 填下 **/bin/bash** 即可。

完整参数：

- **-a stdin:** 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
- **-d:** 后台运行容器，并返回容器ID；
- **-i:** 以交互模式运行容器，通常与 -t 同时使用；
- **-P:** 随机端口映射，容器内部端口**随机**映射到主机的高端口
- **-p:** 指定端口映射，格式为：**主机(宿主)端口:容器端口**
- **-t:** 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
- **--name="nginx-lb":** 为容器指定一个名称；
- **--dns 8.8.8.8:** 指定容器使用的DNS服务器，默认和宿主一致；
- **--dns-search example.com:** 指定容器DNS搜索域名，默认和宿主一致；
- **-h "mars":** 指定容器的hostname；
- **-e username="ritchie":** 设置环境变量；
- **--env-file=[]:** 从指定文件读入环境变量；
- **--cpuset="0-2" or --cpuset="0,1,2":** 绑定容器到指定CPU运行；
- **-m :**设置容器使用内存最大值；
- **--net="bridge":** 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
- **--link=[]:** 添加链接到另一个容器；
- **--expose=[]:** 开放一个端口或一组端口；
- **--volume , -v:** 绑定一个卷

特殊情况下，如需要在centos镜像中使用**systemctl** . 则应添加**--privileged** 并设置[command ]为 **init**。

5 当镜像通过run 启动后，便会载入到一个动态的container(容器)中运行，此时若需要进入终端交互模式：

**sudo docker exec -it [containerID] /bin/bash**

交互模式中，使用  ctrl+p+q退出交互 保持运行,使用 exit命令退出并停止容器。

6 在容器非交互模式下，通过docker  start/stop 命令来启动/停止已部署的容器服务。

7 **docker rm [containerID]** 删除容器

8 **docker rmi [imageID]** 删除镜像

9 **docker cp [YourHostFilePath] [containerID]:[DockerPath]**  将宿主机内的指定文件传输至容器内部的指定地址。

10 **docker logs -f  [containerID]  --tail 10 -t** 查看日志，使用 `-f` 参数后，就可以查看实时日志了，

使用 `--tail` 参数可以精确控制日志的输出行数， `-t` 参数则可以显示日志的输出时间。

#### 6.镜像制作

1  **docker commit [containerID] [ImageName]:[Version]** 将修改后的容器重新打包成镜像

2 **docker commit -a "runoob.com" -m "my apache" a404c6c174a2 mymysql:v1** 将容器a404c6c174a2 保存为新的镜像,并添加提交人信息和说明信息。

**-a** :提交的镜像作者；

 **-c** :使用Dockerfile指令来创建镜像；

 **-m** :提交时的说明文字；

 **-p** :在commit时，将容器暂停。

3 **docker push [ImageID] [repertory_address]**提交镜像到云仓库



## docker容器及镜像清理

To clean up, all unused containers, images, network, and volumes, use the following command.

```
docker system prune
```

**Recommended:** [Learn Docker Technologies for DevOps and Developers 53](http://skillslane.com/offer/learn-docker-technologies-devops-developers/)

To individually delete all the components, use the following commands.

```
docker container prune
docker image prune
docker network prune
docker volume prune
```

For Docker versions below 1.13

To clean up containers, first, you need to clean up your containers. So that all the unwanted images can be deleted without dependency problems.

Delete all Exited Containers

> docker rm $(docker ps -q -f status=exited) 

Delete all Stopped Containers

> docker rm $(docker ps -a -q)
####Delete All Running and Stopped Containers
> docker stop $(docker ps -a -q)
> docker rm $(docker ps -a -q)

#### Delete all “none” Images

```
docker rmi $(docker images | grep "^<none>" | awk '{ print $3 }')
```

#### Delete all Dangling Images

```
sudo docker rmi $(sudo docker images -f "dangling=true" -q)
```



Update Sept. 2016: Docker 1.13: [PR 26108](https://github.com/docker/docker/pull/26108) and [commit 86de7c0](https://github.com/docker/docker/commit/86de7c000f5d854051369754ad1769194e8dd5e1) introduce a few new commands to help facilitate visualizing how much space the docker daemon data is taking on disk and allowing for easily cleaning up "unneeded" excess.

[**docker system prune**](https://docs.docker.com/engine/reference/commandline/system_prune/) will delete ALL dangling data (i.e. In order: containers stopped, volumes without containers and images with no containers). Even unused data, with `-a` option.

You also have:

- [`docker container prune`](https://docs.docker.com/engine/reference/commandline/container_prune/)
- [`docker image prune`](https://docs.docker.com/engine/reference/commandline/image_prune/)
- [`docker network prune`](https://docs.docker.com/engine/reference/commandline/network_prune/)
- [`docker volume prune`](https://docs.docker.com/engine/reference/commandline/volume_prune/)

For *unused* images, use `docker image prune -a` (for removing dangling *and* ununsed images).
Warning: '*unused*' means "images not referenced by any container": be careful before using `-a`.

As illustrated in [A L](https://stackoverflow.com/users/1207596/a-l)'s [answer](https://stackoverflow.com/a/50405599/6309), `docker system prune --all` will remove all *unused* images not just dangling ones... which can be a bit too much.

Combining `docker xxx prune` with the [`--filter` option](https://docs.docker.com/engine/reference/commandline/system_prune/#filtering) can be a great way to limit the pruning ([docker SDK API 1.28 minimum, so docker 17.04+](https://docs.docker.com/develop/sdk/#api-version-matrix))

> The currently supported filters are:

- `until (<timestamp>)` - only remove containers, images, and networks created before given timestamp
- `label` (`label=<key>`, `label=<key>=<value>`, `label!=<key>`, or `label!=<key>=<value>`) - only remove containers, images, networks, and volumes with (or *without*, in case `label!=...` is used) the specified labels.

See "[Prune images](https://docs.docker.com/config/pruning/#prune-images)" for an example.

------

Original answer (Sep. 2016)

I usually do:

```
docker rmi $(docker images --filter "dangling=true" -q --no-trunc)
```

I have an [alias for removing those [dangling images\]](https://github.com/docker/docker/blob/634a848b8e3bdd8aed834559f3b2e0dfc7f5ae3a/man/docker-images.1.md#options)[13](https://github.com/docker/docker/blob/634a848b8e3bdd8aed834559f3b2e0dfc7f5ae3a/man/docker-images.1.md#options): `drmi`

> The `dangling=true` filter finds unused images

That way, any intermediate image no longer referenced by a labelled image is removed.

I do the same **first** for [exited processes (containers)](https://github.com/VonC/b2d/blob/b010ab51974ac7de6162cdcbff795d7b9e84fd67/.bash_aliases#L21)

```
alias drmae='docker rm $(docker ps -qa --no-trunc --filter "status=exited")'
```

As [haridsv](https://stackoverflow.com/users/95750/haridsv) points out [in the comments](https://stackoverflow.com/questions/32723111/how-to-remove-old-and-unused-docker-images/32723127#comment63457575_32723127):

> Technically, **you should first clean up containers before cleaning up images, as this will catch more dangling images and less errors**.

------

[Jess Frazelle (jfrazelle)](https://github.com/jfrazelle) has the [bashrc function](https://github.com/jfrazelle/dotfiles/blob/a7fd3df6ab423e6dd04f27727f653753453db837/.dockerfunc#L8-L11):

```
dcleanup(){
    docker rm -v $(docker ps --filter status=exited -q 2>/dev/null) 2>/dev/null
    docker rmi $(docker images --filter dangling=true -q 2>/dev/null) 2>/dev/null
}
```

------

To remove old images, and not just "unreferenced-dangling" images, you can consider [**docker-gc**](https://github.com/spotify/docker-gc):



```
$ docker images --no-trunc --format '{{.ID}} {{.CreatedSince}} {{.Repository}}' \
    | grep ' months' | awk '{ print $1 }' \
    | xargs --no-run-if-empty docker rmi
```

Example of `/etc/cron.daily/docker-gc` script:

```
#!/bin/sh -e

# Delete all stopped containers (including data-only containers).
docker ps -a -q --no-trunc --filter "status=exited" | xargs --no-run-if-empty docker rm -v

# Delete all tagged images more than a month old
# (will fail to remove images still used).
docker images --no-trunc --format '{{.ID}} {{.CreatedSince}}' | grep ' months' | awk '{ print $1 }' | xargs --no-run-if-empty docker rmi || true

# Delete all 'untagged/dangling' (<none>) images
# Those are used for Docker caching mechanism.
docker images -q --no-trunc --filter dangling=true | xargs --no-run-if-empty docker rmi

# Delete all dangling volumes.
docker volume ls -qf dangling=true | xargs --no-run-if-empty docker volume rm
```



## docker-compose 的安装及使用

### 简单介绍

> Docker Compose 是一个用来定义和运行复杂应用的 Docker 工具。
> 使用 Docker Compose 不再需要使用 shell 脚本来启动容器。(通过 docker-compose.yml 配置)

### 安装

可以通过修改 URL 中的版本，自定义您需要的版本。

- Github源

```
sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

- Daocloud镜像

```
curl -L https://get.daocloud.io/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

### 卸载

```
sudo rm /usr/local/bin/docker-compose
```

### 基础命令

需要在 docker-compose.yml 所在文件夹中执行命令

使用 docker-compose 部署项目的简单步骤

- 停止现有 docker-compose 中的容器：`docker-compose down`
- 重新拉取镜像：`docker-compose pull`
- 后台启动 docker-compose 中的容器：`docker-compose up -d`

下面将介绍 `docker-compose` 子命令的使用。也可以通过运行 `docker-compose --help`来查看这些信息。

- [build](#build)
- [help](#help)
- [kill](#kill)
- [ps](#ps)
- [restart](#restart)
- [run](#run)
- [start](#start)
- [up](#up)
- [logs](#logs)
- [port](#port)
- [pull](#pull)
- [rm](#rm)
- [scale](#scale)
- [stop](#stop)

#### build

```
用法：build [options] [SERVICE...]
选项：
	--force-rm  总是移除构建过程中产生的中间项容器
	--no-cache  构建镜像过程中不使用Cache
	--pull      总是尝试获取更新版本的镜像
```

构建服务并打上`project_service`风格的标签（如：`composetest_db`）。如果你更改了服务的`Dockerfile`或者构建目录下的内容，需要运行`docker-compose build`重新构建服务。

#### help

```
用法：help COMMAND
```

显示命令的帮助信息及用法教程。

#### kill

```
用法：kill [options] [SERVICE...]
选项：
	-s SIGNAL         SIGNAL 是发送给容器的信号量，默认是 SIGKILL
```

通过发送`SIGKILL`信号来强制终止运行中的容器，也可以发送指定的信号量，例如：

```
$ docker-compose kill -s SIGINT
```

#### ps

```
用法：ps [options] [SERVICE...]
选项：
	-q		仅仅显示容器ID
```

列出容器。

#### restart

```
用法：restart [options] [SERVICE...]
选项：
	-t, --timeout TIMEOUT      设置关闭服务的超时时间，单位为秒，默认为10
```

重启服务。

#### run

```
用法：run [options] [-e KEY=VAL...] SERVICE [COMMAND] [ARGS...]
选项：
	-d                    分离模式：在后台运行容器，只打印新的容器名称
	--entrypoint CMD      覆盖镜像的入口点（CMD ...）
	-e KEY=VAL            设置环境变量，可以使用多次
	-u, --user=""         通过指定的用户名或用户id来运行
	--no-deps             不启动link连接的服务
	--rm                  运行结束后移除容器，在分离模式下将被忽略
	-p, --publish=[]      将容器暴露端口映射到主机端口
	--service-ports       通过服务映射到主机的端口执行命令
	-T                    禁用pseudo-tty分配，默认会分配一个TTY
```

对服务运行的命令。例如，以下命令启动web服务并运行bash命令

```
$ docker-compose run web bash
```

`run`命令，将使用服务中已经定义的配置来创建运行一个新的容器。也就是说，如此创建的容器，将会使用相同的挂载卷、容器连接等相同的配置，但它们依旧可以存在差异。

第一个区别是，可以使用`run`命令覆盖服务中指定的运行命令。例如，`web`服务中的配置指定的运行命令为`bash`，那么`docker-compose run web python app.py`将使用`python app.py`来覆盖它。

第二个区别是，`docker-compose run`命令不会创建任何服务配置中指定的端口映射，这样可以防止多个容器映射同一端口的冲突。如果你需要使得服务的端口创建并映射到主机，需要指定`--service-ports`标记，如下：

```
$ docker-compose run --service-ports web python manage.py shell

```

或者可以手动指定端口映射，和使用`docker run`一样，使用`--publish`或`-p`选项：

```
$ docker-compose run --publish 8080:80 -p 2022:22 -p 127.0.0.1:2021:21 web python manage.py shell

```

如果启动一个带有容器连接的服务，`run`命令将首先检查连接到的服务是否已运行，如果是停止状态，将会启动它，直到所有的相关服务都处于正在运行状态，才会执行你创建的命令。例如：

```
$ docker-compose run db psql -h db -U docker

```

这将创建一个与PostgreSQL容器`db`交互服务。

如果你不希望启动相关联容器，可以使用`--no-deps`标记：

```
$ docker-compose run --no-deps web python manage.py shell

```

#### start

```
用法：start [SERVICE...]

```

启动服务中已经存在的容器。

#### up

```
用法：up [options] [SERVICE...]
选项：
	-d                     分离模式：在后台运行容器，只打印新的容器名称
	--no-color             单色输出
	--no-deps              不启动link连接的服务
	--force-recreate       强制重新创建容器，即使镜像没有任何改变。与--no-recreate会冲突
	--no-recreate          如果对应容器已经存在,不重新创建它。与--force-recreate会冲突
	--no-build             不构建镜像，即使缺失
	-t, --timeout TIMEOUT  为容器设置关闭超时时间，单位：秒 (默认为 10)

```

对服务，构建镜像、(重新)创建容器、启动容器。

该命令还将启动任何相关的且没有被启动的服务。

`docker-compose up`命令将显示所有容器的输出，命令结束时，所有容器都将关闭。运行`docker-compose up -d `将在后台启动运行容器。

如果服务中已经存在运行中的容器了，并且在容器创建后更改服务配置或者镜像，`docker-compose up`命令将会停止当前容器（保存挂载卷）并重新构建启动容器。当然，也可以通过`--no-recreate`选项来避免重新构建。

使用`--force-recreate`标记，可以强制停止并重构所有容器。

#### logs

```
用法：logs [options] [SERVICE...]
选项：
	--no-color  单色输出

```

显示服务输出的日志内容。

#### port

```
用法：port [options] SERVICE PRIVATE_PORT
选项：
	--protocol=proto  tcp 或 udp [默认为 tcp]
	--index=index     对应实例服务的第几个容器[默认为 1]

```

打印服务中端口绑定对应的主机端口。

#### pull

```
用法：pull [options] [SERVICE...]
选项：
	--ignore-pull-failures 尽可能拉取服务，忽略拉取失败

```

拉取服务镜像。

#### rm

```
用法：rm [options] [SERVICE...]
选项:
	-f, --force   强制删除，不询问确认信息
	-v            移除容器挂载的卷

```

删除停止的服务容器。

#### scale

```
用法：scale [SERVICE=NUM...]

```

设置一个服务需要运行的容器数量。
参数形式为`service=num`。例如：

```
$ docker-compose scale web=2 worker=3

```

#### stop

```
用法：stop [options] [SERVICE...]
选项：
	-t, --timeout TIMEOUT      设置关闭容器的超时时间

```

停止容器而不移除，可以通`docker-compose start`重新启动。



### 通过 docker-compose.yml 部署应用

我将上面所创建的镜像推送到了阿里云，在此使用它

#### 1.新建 docker-compose.yml 文件

通过以下配置，在运行后可以创建两个站点(只为演示)

```
version: "3"
services:
  web1:
    image: registry.cn-hangzhou.aliyuncs.com/yimo_public/docker-nginx-test:latest
    ports:
      - "4466:80"
  web2:
    image: registry.cn-hangzhou.aliyuncs.com/yimo_public/docker-nginx-test:latest
    ports:
      - "4477:80"
```

此处只是简单演示写法，说明 docker-compose 的方便

#### 2.构建完成，后台运行镜像

```
docker-compose up -d
```

运行后就可以使用 ip+port 访问这两个站点了

#### 3.镜像更新重新部署

```
docker-compose down
docker-compose pull
docker-compose up -d
```



## 详细命名说明

### 1、Docker安装

系统环境：docker最低支持centos7且在64位平台上，内核版本在3.10以上

版本：社区版，企业版（包含了一些收费服务）

官方版安装教程（英文）

> https://docs.docker.com/install/linux/docker-ce/centos/#upgrade-docker-after-using-the-convenience-script

博主版安装教程：

```
# 安装docker
yum install docker
# 启动docker 
systemctl start/status docker 
# 查看docker启动状态
docker version 
```

**配置加速器**

简介：DaoCloud 加速器 是广受欢迎的 Docker 工具，解决了国内用户访问 Docker Hub 缓慢的问题。DaoCloud 加速器结合国内的 CDN 服务与协议层优化，成倍的提升了下载速度。

DaoCloud官网：

> https://www.daocloud.io/mirror#accelerator-doc

```
# 一条命令加速（记得重启docker）
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://95822026.m.daocloud.io
```

------

### 2、Docker基础命令

docker --help（中文注解）

```
Usage:
docker [OPTIONS] COMMAND [arg...]
       docker daemon [ --help | ... ]
       docker [ --help | -v | --version ]
A
self-sufficient runtime for containers.

Options:
  --config=~/.docker              Location of client config files  #客户端配置文件的位置
  -D, --debug=false               Enable debug mode  #启用Debug调试模式
  -H, --host=[]                   Daemon socket(s) to connect to  #守护进程的套接字（Socket）连接
  -h, --help=false                Print usage  #打印使用
  -l, --log-level=info            Set the logging level  #设置日志级别
  --tls=false                     Use TLS; implied by--tlsverify  #
  --tlscacert=~/.docker/ca.pem    Trust certs signed only by this CA  #信任证书签名CA
  --tlscert=~/.docker/cert.pem    Path to TLS certificate file  #TLS证书文件路径
  --tlskey=~/.docker/key.pem      Path to TLS key file  #TLS密钥文件路径
  --tlsverify=false               Use TLS and verify the remote  #使用TLS验证远程
  -v, --version=false             Print version information and quit  #打印版本信息并退出

Commands:
    attach    Attach to a running container  #当前shell下attach连接指定运行镜像
    build     Build an image from a Dockerfile  #通过Dockerfile定制镜像
    commit    Create a new image from a container's changes  #提交当前容器为新的镜像
    cp    Copy files/folders from a container to a HOSTDIR or to STDOUT  #从容器中拷贝指定文件或者目录到宿主机中
    create    Create a new container  #创建一个新的容器，同run 但不启动容器
    diff    Inspect changes on a container's filesystem  #查看docker容器变化
    events    Get real time events from the server#从docker服务获取容器实时事件
    exec    Run a command in a running container#在已存在的容器上运行命令
    export    Export a container's filesystem as a tar archive  #导出容器的内容流作为一个tar归档文件(对应import)
    history    Show the history of an image  #展示一个镜像形成历史
    images    List images  #列出系统当前镜像
    import    Import the contents from a tarball to create a filesystem image  #从tar包中的内容创建一个新的文件系统映像(对应export)
    info    Display system-wide information  #显示系统相关信息
    inspect    Return low-level information on a container or image  #查看容器详细信息
    kill    Kill a running container  #kill指定docker容器
    load    Load an image from a tar archive or STDIN  #从一个tar包中加载一个镜像(对应save)
    login    Register or log in to a Docker registry#注册或者登陆一个docker源服务器
    logout    Log out from a Docker registry  #从当前Docker registry退出
    logs    Fetch the logs of a container  #输出当前容器日志信息
    pause    Pause all processes within a container#暂停容器
    port    List port mappings or a specific mapping for the CONTAINER  #查看映射端口对应的容器内部源端口
    ps    List containers  #列出容器列表
    pull    Pull an image or a repository from a registry  #从docker镜像源服务器拉取指定镜像或者库镜像
    push    Push an image or a repository to a registry  #推送指定镜像或者库镜像至docker源服务器
    rename    Rename a container  #重命名容器
    restart    Restart a running container  #重启运行的容器
    rm    Remove one or more containers  #移除一个或者多个容器
    rmi    Remove one or more images  #移除一个或多个镜像(无容器使用该镜像才可以删除，否则需要删除相关容器才可以继续或者-f强制删除)
    run    Run a command in a new container  #创建一个新的容器并运行一个命令
    save    Save an image(s) to a tar archive#保存一个镜像为一个tar包(对应load)
    search    Search the Docker Hub for images  #在docker
hub中搜索镜像
    start    Start one or more stopped containers#启动容器
    stats    Display a live stream of container(s) resource usage statistics  #统计容器使用资源
    stop    Stop a running container  #停止容器
    tag         Tag an image into a repository  #给源中镜像打标签
    top       Display the running processes of a container #查看容器中运行的进程信息
    unpause    Unpause all processes within a container  #取消暂停容器
    version    Show the Docker version information#查看容器版本号
    wait         Block until a container stops, then print its exit code  #截取容器停止时的退出状态值

Run 'docker COMMAND --help' for more information on a command.  #运行docker命令在帮助可以获取更多信息
docker search  hello-docker  # 搜索hello-docker的镜像
docker search centos # 搜索centos镜像
docker pull hello-docker # 获取centos镜像
docker run  hello-world   #运行一个docker镜像，产生一个容器实例（也可以通过镜像id前三位运行）
docker image ls  # 查看本地所有镜像
docker images  # 查看docker镜像
docker image rmi hello-docker # 删除centos镜像
docker ps  #列出正在运行的容器（如果创建容器中没有进程正在运行，容器就会立即停止）
docker ps -a  # 列出所有运行过的容器记录
docker save centos > /opt/centos.tar.gz  # 导出docker镜像至本地
docker load < /opt/centos.tar.gz   #导入本地镜像到docker镜像库
docker stop  `docker ps -aq`  # 停止所有正在运行的容器
docker  rm `docker ps -aq`    # 一次性删除所有容器记录
docker rmi  `docker images -aq`   # 一次性删除所有本地的镜像记录
```

#### 2.1 启动容器的两种方式

容器是运行应用程序的，所以必须得先有一个操作系统为基础

1、基于镜像新建一个容器并启动

```
# 1. 后台运行一个docker
docker run -d centos /bin/sh -c "while true;do echo 正在运行; sleep 1;done"
    # -d  后台运行容器
    # /bin/sh  指定使用centos的bash解释器
    # -c 运行一段shell命令
    # "while true;do echo 正在运行; sleep 1;done"  在linux后台，每秒中打印一次正在运行
docker ps  # 检查容器进程
docker  logs  -f  容器id/名称  # 不间断打印容器的日志信息 
docker stop centos  # 停止容器

# 2. 启动一个bash终端,允许用户进行交互
docker run --name mydocker -it centos /bin/bash  
    # --name  给容器定义一个名称
    # -i  让容器的标准输入保持打开
    # -t 让Docker分配一个伪终端,并绑定到容器的标准输入上
    # /bin/bash 指定docker容器，用shell解释器交互
```

当利用docker run来创建容器时，Docker在后台运行的步骤如下：

> 1. 检查本地是否存在指定的镜像，不存在就从公有仓库下载
> 2. 利用镜像创建并启动一个容器
> 3. 分配一个文件系统，并在只读的镜像层外面挂在一层可读写层
> 4. 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
> 5. 从地址池配置一个ip地址给容器
> 6. 执行用户指定的应用程序
> 7. 执行完毕后容器被终止

2、将一个终止状态(stopped)的容器重新启动

```
[root@localhost ~]# docker ps -a  # 先查询记录
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                        PORTS                    NAMES
ee92fcf6f32d        centos              "/bin/bash"              4 days ago          Exited (137) 3 days ago                                kickass_raman

[root@localhost ~]# docker start ee9  # 再启动这个容器
ee9

[root@localhost ~]# docker exec -it  ee9 /bin/bash  # 进入容器交互式界面
[root@ee92fcf6f32d /]#   # 注意看用户名，已经变成容器用户名
```

#### 2.2 提交创建自定义镜像

```
# 1.我们进入交互式的centos容器中，发现没有vim命令
    docker run -it centos
# 2.在当前容器中，安装一个vim
    yum install -y vim
# 3.安装好vim之后，exit退出容器
    exit
# 4.查看刚才安装好vim的容器记录
    docker container ls -a
# 5.提交这个容器，创建新的image
    docker commit 059fdea031ba chaoyu/centos-vim
# 6.查看镜像文件
    docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
chaoyu/centos-vim   latest              fd2685ae25fe        5 minutes ago   
```

#### 2.3 外部访问容器

容器中可以运行网络应用，但是要让外部也可以访问这些应用，可以通过-p或-P参数指定端口映射。

```
docker run -d -P training/webapp python app.py
  # -P 参数会随机映射端口到容器开放的网络端口

# 检查映射的端口
docker ps -l
CONTAINER ID        IMAGE               COMMAND             CREATED            STATUS              PORTS                     NAMES
cfd632821d7a        training/webapp     "python app.py"     21 seconds ago      Up 20 seconds       0.0.0.0:32768->5000/tcp   brave_fermi
#宿主机ip:32768 映射容器的5000端口

# 查看容器日志信息
docker logs -f cfd  # #不间断显示log

# 也可以通过-p参数指定映射端口
docker run -d -p 9000:5000 training/webapp python app.py
```

打开浏览器访问服务器的9000端口， 内容显示 Hello world！表示正常启动

(如果访问失败的话，检查自己的防火墙，以及云服务器的安全组)

------

### 3、利用dockerfile定制镜像

镜像是容器的基础，每次执行docker run的时候都会指定哪个镜像作为容器运行的基础。我们之前的例子都是使用来自docker hub的镜像，直接使用这些镜像只能满足一定的需求，当镜像无法满足我们的需求时，就得自定制这些镜像。

> 镜像的定制就是定制每一层所添加的配置、文件。如果可以吧每一层修改、安装、构建、操作的命令都写入到一个脚本，用脚本来构建、定制镜像，这个脚本就是dockerfile。
>
> Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令 构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

参数详解

```
FROM scratch #制作base image 基础镜像，尽量使用官方的image作为base image
FROM centos #使用base image
FROM ubuntu:14.04 #带有tag的base image

LABEL version=“1.0” #容器元信息，帮助信息，Metadata，类似于代码注释
LABEL maintainer=“yc_uuu@163.com"

#对于复杂的RUN命令，避免无用的分层，多条命令用反斜线换行，合成一条命令！
RUN yum update && yum install -y vim 
    Python-dev #反斜线换行
RUN /bin/bash -c "source $HOME/.bashrc;echo $HOME”

WORKDIR /root #相当于linux的cd命令，改变目录，尽量使用绝对路径！！！不要用RUN cd
WORKDIR /test # 如果没有就自动创建
WORKDIR demo # 再进入demo文件夹
RUN pwd     # 打印结果应该是/test/demo

ADD and COPY 
ADD hello /  # 把本地文件添加到镜像中，吧本地的hello可执行文件拷贝到镜像的/目录
ADD test.tar.gz /  # 添加到根目录并解压

WORKDIR /root
ADD hello test/  # 进入/root/ 添加hello可执行命令到test目录下，也就是/root/test/hello 一个绝对路径
COPY hello test/  # 等同于上述ADD效果

ADD与COPY
   - 优先使用COPY命令
    -ADD除了COPY功能还有解压功能
添加远程文件/目录使用curl或wget

ENV # 环境变量，尽可能使用ENV增加可维护性
ENV MYSQL_VERSION 5.6 # 设置一个mysql常量
RUN yum install -y mysql-server=“${MYSQL_VERSION}” 
```

进阶知识(了解)

```
VOLUME and EXPOSE 
存储和网络

RUN and CMD and ENTRYPOINT
RUN：执行命令并创建新的Image Layer
CMD：设置容器启动后默认执行的命令和参数
ENTRYPOINT：设置容器启动时运行的命令

Shell格式和Exec格式
RUN yum install -y vim
CMD echo ”hello docker”
ENTRYPOINT echo “hello docker”

Exec格式
RUN [“apt-get”,”install”,”-y”,”vim”]
CMD [“/bin/echo”,”hello docker”]
ENTRYPOINT [“/bin/echo”,”hello docker”]


通过shell格式去运行命令，会读取$name指令，而exec格式是仅仅的执行一个命令，而不是shell指令
cat Dockerfile
    FROM centos
    ENV name Docker
    ENTRYPOINT [“/bin/echo”,”hello $name”]#这个仅仅是执行echo命令，读取不了shell变量
    ENTRYPOINT  [“/bin/bash”,”-c”,”echo hello $name"]

CMD
容器启动时默认执行的命令
如果docker run指定了其他命令(docker run -it [image] /bin/bash )，CMD命令被忽略
如果定义多个CMD，只有最后一个执行

ENTRYPOINT
让容器以应用程序或服务形式运行
不会被忽略，一定会执行
最佳实践：写一个shell脚本作为entrypoint
COPY docker-entrypoint.sh /usr/local/bin
ENTRYPOINT [“docker-entrypoint.sh]
EXPOSE 27017
CMD [“mongod”]

[root@master home]# more Dockerfile
FROm centos
ENV name Docker
#CMD ["/bin/bash","-c","echo hello $name"]
ENTRYPOINT ["/bin/bash","-c","echo hello $name”]
```

------

### 4、发布到仓库

#### 4.1 docker hub共有镜像发布

docker提供了一个类似于github的仓库docker hub，官方网站（需注册使用）

> https://hub.docker.com/

```
# 注册docker id后，在linux中登录dockerhub
    docker login

# 注意要保证image的tag是账户名，如果镜像名字不对，需要改一下tag
    docker tag chaoyu/centos-vim peng104/centos-vim
    # 语法是：docker tag   仓库名   peng104/仓库名

# 推送docker image到dockerhub
    docker push peng104/centps-cmd-exec:latest

# 去dockerhub中检查镜像
# 先删除本地镜像，然后再测试下载pull 镜像文件
    docker pull peng104/centos-entrypoint-exec
```

#### 4.2 私有仓库

docker hub 是公开的，其他人也是可以下载，并不安全，因此还可以使用docker registry官方提供的私有仓库

用法详解：

> https://yeasy.gitbooks.io/docker_practice/repository/registry.html

```
# 1.下载一个docker官方私有仓库镜像
    docker pull registry
# 2.运行一个docker私有容器仓库
docker run -d -p 5000:5000 -v /opt/data/registry:/var/lib/registry  registry
    -d 后台运行 
    -p  端口映射 宿主机的5000:容器内的5000
    -v  数据卷挂载  宿主机的 /opt/data/registry :/var/lib/registry 
    registry  镜像名
    /var/lib/registry  存放私有仓库位置
# Docker 默认不允许非 HTTPS 方式推送镜像。我们可以通过 Docker 的配置选项来取消这个限制
# 3.修改docker的配置文件，让他支持http方式，上传私有镜像
    vim /etc/docker/daemon.json 
    # 写入如下内容
    {
        "registry-mirrors": ["http://f1361db2.m.daocloud.io"],
        "insecure-registries":["192.168.11.37:5000"]
    }
# 4.修改docker的服务配置文件
    vim /lib/systemd/system/docker.service
# 找到[service]这一代码区域块，写入如下参数
    [Service]
    EnvironmentFile=-/etc/docker/daemon.json
# 5.重新加载docker服务
    systemctl daemon-reload
# 6.重启docker服务
    systemctl restart docker
    # 注意:重启docker服务，所有的容器都会挂掉

# 7.修改本地镜像的tag标记，往自己的私有仓库推送
    docker tag docker.io/peng104/hello-world-docker 192.168.11.37:5000/peng-hello
    # 浏览器访问http://192.168.119.10:5000/v2/_catalog查看仓库
# 8.下载私有仓库的镜像
    docker pull 192.168.11.37:5000/peng-hello
```

------

### 5、实例演示

编写dockerfile，构建自己的镜像，运行flask程序。

确保app.py和dockerfile在同一个目录！

```
# 1.准备好app.py的flask程序
    [root@localhost ~]# cat app.py
    from flask import Flask
    app=Flask(__name__)
    @app.route('/')
    def hello():
        return "hello docker"
    if __name__=="__main__":
        app.run(host='0.0.0.0',port=8080)
    [root@master home]# ls
    app.py  Dockerfile

# 2.编写dockerfile
    [root@localhost ~]# cat Dockerfile
    FROM python:2.7
    LABEL maintainer="温而新"
    RUN pip install flask
    COPY app.py /app/
    WORKDIR /app
    EXPOSE 8080
    CMD ["python","app.py"]

# 3.构建镜像image,找到当前目录的Dockerfile，开始构建
    docker build -t peng104/flask-hello-docker .

# 4.查看创建好的images
    docker image ls

# 5.启动此flask-hello-docker容器，映射一个端口供外部访问
    docker run -d -p 8080:8080 peng104/flask-hello-docker

# 6.检查运行的容器
    docker container ls

# 7.推送这个镜像到私有仓库
    docker tag  peng104/flask-hello-docker   192.168.11.37:5000/peng-flaskweb
    docker push 192.168.11.37:5000/peng-flaskweb
```



## 补充一些用法

### 挂载宿主机目录

docker可以支持把一个宿主机上的目录挂载到镜像里。

`docker run -it -v /home/dock/Downloads:/usr/Downloads ubuntu64 /bin/bash`
通过-v参数，冒号前为宿主机目录，必须为绝对路径，冒号后为镜像内挂载的路径。

现在镜像内就可以共享宿主机里的文件了。

默认挂载的路径权限为读写。如果指定为只读可以用：ro
`docker run -it -v /home/dock/Downloads:/usr/Downloads:ro ubuntu64 /bin/bash`

docker还提供了一种高级的用法。叫数据卷。

数据卷：“其实就是一个正常的容器，专门用来提供数据卷供其它容器挂载的”。感觉像是由一个容器定义的一个数据挂载信息。其他的容器启动可以直接挂载数据卷容器中定义的挂载信息。

看示例：
`docker run -v /home/dock/Downloads:/usr/Downloads --name dataVol ubuntu64 /bin/bash`
创建一个普通的容器。用--name给他指定了一个名（不指定的话会生成一个随机的名子）。

再创建一个新的容器，来使用这个数据卷。
`docker run -it --volumes-from dataVol ubuntu64 /bin/bash --volumes-from用来指定要从哪个数据卷来挂载数据。`

 

生成的docker镜像保存下来：docker save -o test.tar test:1.0
 导入docker镜像：docker load -i test.tar



### Docker容器内外互相拷贝数据

```
从容器内拷贝文件到主机上
docker cp <containerId>:/file/path/within/container /host/path/target 

从主机上拷贝文件到容器内
参考自：
http://stackoverflow.com/questions/22907231/copying-files-from-host-to-docker-container
1.用-v挂载主机数据卷到容器内
docker run -v /path/to/hostdir:/mnt $container
在容器内拷贝
cp /mnt/sourcefile /path/to/destfile
2.直接在主机上拷贝到容器物理存储系统
A. 获取容器名称或者id :
$ docker ps
B. 获取整个容器的id
$ docker inspect -f   '{{.Id}}'  步骤A获取的名称或者id
C. 在主机上拷贝文件:
$ sudo cp path-file-host /var/lib/docker/aufs/mnt/FULL_CONTAINER_ID/PATH-NEW-FILE 
或者
$ sudo cp path-file-host /var/lib/docker/devicemapper/mnt/123abc<<id>>/rootfs/root

例子：
$ docker ps
CONTAINER ID      IMAGE    COMMAND       CREATED      STATUS       PORTS        NAMES
d8e703d7e303   solidleon/ssh:latest      /usr/sbin/sshd -D                      cranky_pare

$ docker inspect -f   '{{.Id}}' cranky_pare
or 
$ docker inspect -f   '{{.Id}}' d8e703d7e303
d8e703d7e3039a6df6d01bd7fb58d1882e592a85059eb16c4b83cf91847f88e5

$ sudo cp file.txt /var/lib/docker/aufs/mnt/**d8e703d7e3039a6df6d01bd7fb58d1882e592a85059eb16c4b83cf91847f88e5

3.用输入输出符
docker run -i ubuntu /bin/bash -c 'cat > /path/to/container/file' < /path/to/host/file/

或者

docker exec -it <container_id> bash -c 'cat > /path/to/container/file' < /path/to/host/file/
```

**总结一下**

> 从主机复制到容器`sudo docker cp host_path containerID:container_path`
>
> 从容器复制到主机`sudo docker cp containerID:container_path host_path`
>
> 容器ID的查询方法想必大家都清楚:`docker ps -a`



### docker 运行容器时为容器起别名

>docker run --name=mydemo -p  -d 2222:80 imagename
>
>--name: 指定容器名称
>
>-p:指定容器端口号
>
>-d:指定容器后台运行



### docker一些有用的清理命令

以下命令参考自这篇[文章](http://blog.loof.fr/2016/05/docker-cleanup.html#)：

（1）清除已经终止的`container`：

```
docker rm -v $(docker ps --filter status=exited -q)
```

（2）清除已经没用的`volume`：

```
docker volume rm $(docker volume ls -q -f 'dangling=true')
```

（3）清除已经没用的`image`：

```
docker rmi $(docker images -f "dangling=true" -q) 
```

（4）清除所有的`container`（包括正在运行的和已经退出的）：

```
docker rm -f $(docker ps -a | awk 'NR > 1 {print $1}')
```



### 在docker container中执行命令的脚本

下面脚本的功能是循环地在各个`container`中执行命令：

```
#!/bin/bash -x

for i in {1..2}
do
        docker exec -i hammerdb_net${i} bash <<-EOF
        su oracle
        source /tmp/ora_env
        cd /data/oracle/tablespaces/
        rm -f *.html
        ./create_awr.sh
        mv awr.html awr_${i}.html
        EOF
done
```

需要注意的是在`do`和`done`之间应该使用`tab`而不是空格。

