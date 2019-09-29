# 02镜像的制作与导入导出
## 镜像的制作

- 通过 Dockerfile 制作镜像（高级常用做法）
- 基于容器制作镜像（初学者做法）
    - 首先将别人做好的镜像跑成容器，做了修改以后（例如别人做好了一个 busybox 镜像，然后我用这个镜像创建了一个容器，在容器中我在根目录创建一个目录 test）
    - `docker commit -p b1`暂停容器b1并生成镜像，这个命令的末尾还能增加参数来指明仓库和标签
    - `docker login`登陆服务器
    - `docker push`推送镜像
- Docker Hub automated builds（也可以使用阿里云容器镜像服务的自动构建，本质上就是通过 Dockerfile 制作镜像）

## 镜像的导入导出

- docker save -o myimages.gz 镜像1 镜像2
- 转移文件 myimages.gz
- docker load -i myimages.gz
