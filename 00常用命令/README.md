# 00常用命令
## 镜像相关的常用命令
| 命令 | 解释 | 备注 |
| ----- | ----- | ----- |
| docker search | 搜索镜像 | `docker search -s 10 java` *从 Docker Hub 查找所有镜像名包含 java，并且收藏数大于10的镜像* |
| docker pull | 从指定的registry中下载镜像 | `docker pull maimidoudou6-registry.cn-beijing.cr.aliyuncs.com/base/centos:7.6.1810` |
| docker images | 列出本地所有镜像 |  |
| docker rmi | 删除镜像 | `docker rmi maimidoudou6-registry.cn-beijing.cr.aliyuncs.com/base/centos:7.6.1810` |
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
| docker ps -a | 列出所有的容器 | *非常常用* |
| docker run --name b1 -it --rm -d busybox:latest | 使用 busybox:latest 镜像创建容器并运行，该容器的名字为 b1 并且是交互式运行，当关闭容器时就删除该容器，后台运行 | *非常常用* |
| docker inspect b1 | 查看容器 b1 的信息例如 IP 地址等 | *非常常用* |
| docker inspect -f {{.Config}} b1 | 查看容器 b1 的信息中 Config 的信息 |  |
| docker exec -it b1 /bin/sh | 命令行登陆进 b1 容器中，其中 /bin/sh 可以灵活运用从而实现高级操作，exit退出后不会关闭容器 | *非常常用* |
| docker attach b1 | 直接登陆进 b1 容器中，即容器运行的主程序中，如果此时容器运行的主程序是 /bin/sh 则 exit 之后会直接关闭容器，因为容器的运行是由主程序支撑起来的 | *非常常用* |
| docker tag | 给镜像打标签，打标签不是覆盖，同一个镜像可以有多个标签<br>原始镜像有标签则 docker tag 旧镜像:旧标签 新镜像:新标签<br>原始镜像无标签则 docker tag ID号 新镜像:新标签 | *非常常用* |

