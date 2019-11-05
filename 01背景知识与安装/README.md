# 01背景知识与安装
## 背景知识
### 主机级虚拟化

- Type-I：硬件虚拟化引擎（直接运行于硬件系统之上的裸机管理程序）
- Type-II：以现有操作系统之上的应用程序之一的方式运行（virtualbox）

| Linux Namespaces | 系统调用参数 | 隔离内容 | 内核版本 |
| ------ | ------ | ------ | ------ |
| UTS | CLONE_NEWUTS | 主机名和域名 | 2.6.19 |
| IPC | CLONE_NEWIPC | 信号量、消息队列和共享内存 | 2.6.19 |
| PID | CLONE_NEWPID | 进程编号 | 2.6.24 |
| Network | CLONE_NEWNET | 网络设备、网络栈、端口等 | 2.6.29 |
| Mount | CLONE_NEWNS | 挂载点（文件系统） | 2.4.19 |
| User | CLONE_NEWUSER | 用户和用户组 | 3.8（版本过高，所以不能使用 centos6） |

### 容器级虚拟化

通过 Control Group 实现用户空间的资源分配，镜像是静态的，容器是动态的所以具有生命周期

## 安装 Docker
### Linux 安装 Docker CE

1. `wget https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo`
2. 命令行 `sed -i 's/https:\/\/download.docker.com\//https:\/\/mirrors.tuna.tsinghua.edu.cn\/docker-ce\//g' docker-ce.repo` 修改文件，即替换 `https://download.docker.com/` 为 `https://mirrors.tuna.tsinghua.edu.cn/docker-ce/`
3. 将文件移动到 /etc/yum.repos.d/
4. 命令行 `yum install docker-ce`
5. 创建配置文件 /etc/docker/daemon.json 并输入如下内容：
    ```
    {
        "registry-mirrors": ["https://registry.docker-cn.com"]
    }
    ```
6. 命令行 `systemctl restart docker.service` 启动服务
7. 命令行 `docker --version` 检查是否成功
8. 命令行 `systemctl enable docker.service` 开启启动
9. 为了使当前用户具有使用 docker 命令的权限，需要依次运行如下命令：
    ```
    sudo groupadd docker
    sudo gpasswd -a ${USER} docker
    systemctl restart docker
    newgrp - docker
    ```

### Linux 安装 Docker Compose

Docker Compose 是 Docker 的单机编排工具，它的安装需要在安装了 Docker CE 的基础上依次运行如下命令：

```
sudo curl -L https://github.com/docker/compose/releases/download/1.19.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

### Mac 安装 Docker

Google 搜索 docker for mac，通常安装的是套装，既包含 Docker CE 又包含 Docker Compose 等组件
