---
title: docker使用说明
date: 2019-10-30 21:12:14
updated: 2019-11-1 17:01:54 #手动添加更新时间
top: 2
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

作者：爱睡觉的树

链接：

https://www.jianshu.com/p/a84e8cf33b34

来源：简书

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

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

