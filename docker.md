# Docker
镜像-可执行的包（包括应用运行需要的一切条件）

容器-镜像的运行时态

```
docker ps #查看运行中的容器
docker image ls # 查看docker 本地镜像
```

- 栈 定义服务沟通
- 服务 容器在生产中的行为
- 容器 应用

`Dockerfile`定义容器的环境

```
docker build --tag=friendlyhello #创建docker镜像

docker run -p 4000:80 friendlyhello #启动镜像应用/容器
docker run -d -p 4000:80 friendlyhello # 守护进程启动

docker tag image(本地镜像名) username/repository:tag # 关联本地和仓库镜像和版本

docker push username/repository:tag # 发布

docker run -p 4000:80 username/repository:tag # 运行仓库项目
#docker push alfredcarl165/get-started:part1
```

## `docker-compose.yml` 定义doker容器生产中的行为
```
docker swarm init # 新app负载均衡中先运行

docker stack deploy -c docker-compose.yml getstartedlab # 运行app，应用compose.yml 指导

docker service ls # 查看服务运行状况

docker stack services getstartedlab #查看指定的服务

docker service ps getstartedlab_web # 查看task
```

服务中的每个容器被称作 `task`

```
# take down app
docker stack rm getstartedlab

# take down swarm
docker swarm leave --force
```

## swarm
多容器、多机器应用使用多机器的docker化的集群成为swarm

swarm中的机器成为 node

swarm manager 是集群中唯一能够运行命令的机器，或者允许其他机器加入swarm 成为 worker


```
# create VM
docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2

# check machine
docker-machine ls

# init swarm add node
docker-machine ssh myvm1 "docker swarm init --advertise-addr ipAddress"

# 添加工作节点
dokcer-machine ssh myvm2 "docker swarm join --token SWMTKN-1-59cwdwii5jpvkytz15oh00z3cjzcdnfhpm07itfs3jpul60k3p-alo9t9yezhaqxigm1j9g6lbdr 192.168.99.100:2377"

# 添加管理节点
docker-machine ssh myvm1 "docker swarm join-token manager "

# 查看节点
docker-machine ssh myvm1 "docker node ls"

# 配置本地shell，配置过后不用包装ssh命令
docker-machine env myvm1
eval $(docker-machine env myvm1)

## 集群部署
docker-machine scp docker-compose.yml myvm1
dokcer-machine ssh myvm1 "docker stack deploy -c docker-compose.yml getstartedlab"

# 查看集群
docker-machine ssh myvm1 "docker stack ps getstartedlab"

```
2376、2377 端口有特殊用途不能占用

```
virtualbox 安装完成后启动报错
RC=-1908

解决方法：
1. 将Virtualbox 推出
2. 运行以下命令
# mac 终端运行
sudo spctl --master-disable
```





"Alexandra Sifferlin, Medium" <editors-picks@medium.com>
