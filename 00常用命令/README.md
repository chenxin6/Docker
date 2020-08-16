# 00常用命令
## 镜像相关的常用命令

| 命令 | 解释 | 备注 |
| ----- | ----- | ----- |
| docker search | 搜索镜像 | `docker search -s 10 java` *从 Docker Hub 查找所有镜像名包含 java，并且收藏数大于10的镜像* |
| docker pull | 从指定的registry中下载镜像 | `docker pull maimidoudou6-registry.cn-beijing.cr.aliyuncs.com/base/centos:7.6.1810` |
| docker images | 列出本地所有镜像 |  |
| docker rmi | 删除镜像 | `docker rmi maimidoudou6-registry.cn-beijing.cr.aliyuncs.com/base/centos:7.6.1810` |
| docker -H 192.168.56.2:2375 ps -a | 远程操作查看192.168.56.2宿主机上的所有容器，192.168.56.2宿主机上需要更改配置详见`03网络相关`中的`其他` | 高级用法 |
| docker image ls -q | 列出本地所有镜像的 ID | `rpm -qa | grep openjdk | xargs -I {} sudo rpm -e --nodeps {}`这个命令是删除所有安装了的 openjdk，可以模仿这个进行所有镜像的删除 |

## 容器相关的常用命令

| 命令 | 解释 | 备注 |
| ----- | ----- | ----- |
| docker container create | 创建新容器 |  |
| docker container start | 启动未启动的容器 |  |
| docker container stop | 关闭运行的容器 |  |
| docker container rm | 删除容器 |  |
| docker container pause | 暂停容器 |  |
| docker container unpause | 取消暂停容器 |  |
| docker container top | 展示运行的容器的信息 |  |
| docker container ls -a | 列出所有的容器 |  |
| docker container cp | 将宿主机文件拷贝进容器，或者将容器中的文件拷贝进宿主机 |  |
| docker ps -a | 列出所有的容器，`docker ps -aq`列出所有容器的 ID | *非常常用* |
| docker run --name b1 -it --rm -d busybox:latest | 使用 busybox:latest 镜像创建容器并运行，该容器的名字为 b1 并且是交互式运行，当关闭容器时就删除该容器，后台运行 | *非常常用* |
| docker inspect b1 | 查看容器 b1 的信息例如 IP 地址等 | *非常常用* |
| docker inspect -f {{.Config}} b1 | 查看容器 b1 的信息中 Config 的信息 |  |
| docker exec -it b1 /bin/sh | 命令行登陆进 b1 容器中，其中 /bin/sh 可以灵活运用从而实现高级操作，exit退出后不会关闭容器 | *非常常用* |
| docker attach b1 | 直接登陆进 b1 容器中，即容器运行的主程序中，如果此时容器运行的主程序是 /bin/sh 则 exit 之后会直接关闭容器，因为容器的运行是由主程序支撑起来的，如果想不关闭容器可以使用 ctrl+p+q | *非常常用* |
| docker tag | 给镜像打标签，打标签不是覆盖，同一个镜像可以有多个标签<br>原始镜像有标签则 docker tag 旧镜像:旧标签 新镜像:新标签<br>原始镜像无标签则 docker tag ID号 新镜像:新标签 | *非常常用* |

## 容器网络相关的常用命令

| 命令 | 解释 | 备注 |
| ----- | ----- | ----- |
| `docker run --name b1 --network container:b2 --rm busybox:latest` | 使用 busybox:latest 镜像创建容器并运行，该容器的名字为 b1 并且与容器 b2 共享网络命名空间，当关闭容器时就删除该容器 | *非常常用* |
| `docker run --name b1 --network host --rm busybox:latest` | 使用 busybox:latest 镜像创建容器并运行，该容器的名字为 b1 并且与宿主机共享网络命名空间，当关闭容器时就删除该容器 | *非常常用* |
| `docker run --name b1 -it --network bridge --hostname b1.test.com busybox:latest` | 使用 busybox:latest 镜像创建容器并运行，该容器的名字为 b1 并且是交互式运行，网络模式为桥接模式并指定主机名为 b1.test.com，如果想加本地主机名解析则添加`--add-host "www.baidu.com:182.61.200.7"`，如果想指定 dns 服务器则添加`--dns 8.8.8.8`，如果想加搜索域则添加`--dns-search ilinux.io` | *非常常用* |
| `docker run --name b1 -p 10080:80 registry.cn-hangzhou.aliyuncs.com/maimidoudou6/mybusybox` | 使用 registry.cn-hangzhou.aliyuncs.com/maimidoudou6/mybusybox 镜像创建容器并运行，该容器的名字为 b1 利用 NAT 网络地址暴露端口，将容器的端口80映射宿主机端口10080， | *非常常用* |

- `-p 80`将容器端口80映射宿主机所有地址的一个动态端口
- `-p 80:90`将容器端口90映射宿主机所有地址的80端口
- `-p 192.168.1.1::80`将容器端口80映射宿主机192.168.1.1的一个动态端口
- `-p 192.168.1.1:90:80`将容器端口80映射宿主机192.168.1.1的90端口

动态端口即随机端口，具体的映射结果可以通过命令`docker port`查看，注意宿主机一个端口已经被一个容器映射所占用就不能再建立一个容器映射宿主机的该端口

- `iptables -t nat -vnL`宿主机运行该命令可查看映射结果
- `netstat -nlp`容器中运行该命令查看端口监听情况

## 容器存储卷相关的常用命令

| 命令 | 解释 | 备注 |
| ----- | ----- | ----- |
| `docker run --name b1 -it -v /data/volume/b1:/data busybox:latest` | 使用 busybox:latest 镜像创建容器并运行，该容器的名字为 b1 并且是交互式运行，容器中的 /data 与宿主机 /data/volume/b1 相关联，**注意容器中原本是没有 /data 这个目录的** | *非常常用* |
| `docker run --name b1 -it -v /data busybox:latest` | 使用 busybox:latest 镜像创建容器并运行，该容器的名字为 b1 并且是交互式运行，容器中的 /data 与 docker 管理的某个卷相关联，**注意容器中原本是没有 /data 这个目录的** |  |
| `docker run --name b2 -it --volumes-from b1 busybox:latest` | 使用 busybox:latest 镜像创建容器并运行，该容器的名字为 b2 并且是交互式运行，容器 b2 和容器 b1 使用相同的存储卷 | *避免了重复性操作* |

