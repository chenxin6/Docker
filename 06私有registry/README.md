# 私有 registry
## 利用 Harbor 构建私有仓库

```
wget https://storage.googleapis.com/harbor-releases/release-1.8.0/harbor-offline-installer-v1.8.0.tgz
tar -zxvf harbor-offline-installer-v1.8.0.tgz
cd harbor/
```

给 harbor.yml 修改和添加如下内容：

```
hostname: 192.168.126.27
```

由于默认情况下搭建的 registry 是不支持 https，而 Docker 客户端默认是要求必须使用 https，所以需要修改 Docker 客户端的配置，即在`/etc/docker/daemon.json`添加下内容：

```
"insecure-registries": ["192.168.126.27"]
```

然后重启服务

```
systemctl restart docker.service
```

然后切换用户到 root 并在之前解压缩的文件夹中执行脚本

```
su
sh install.sh
```

自此便可正常使用了，浏览器输入`http://192.168.126.27`可进入前端界面，初始用户为 admin 密码为 Harbor12345，登陆进去后可以看到这个用户有一个默认的命名空间 library ，所以如果想推送镜像则依次运行如下命令

```
# 密码是Harbor12345
docker login --username=admin 192.168.126.27
# 给一个镜像打上新标签
docker tag goharbor/nginx-photon:v1.8.0 192.168.126.27/library/nginx-photon:latest
# 推送
docker push 192.168.126.27/library/nginx-photon
```

如果想使用阿里云的 OSS 作为存储镜像的地方可以修改文件后重启整个集群。首先是修改文件`/home/ioc/dockercompose/harbor/harbor/common/config/registry/config.yml`，其中 harbor-offline-installer-v1.8.0.tgz 是下载在`/home/ioc/dockercompose/harbor/`这个目录下，将文件中原来的 storage 内容都删除然后输入如下内容：

```
storage:
  oss:
    accesskeyid: xxxxxx
    accesskeysecret: xxxxxx
    region: oss-cn-hangzhou
    endpoint: maimidoudou6firstbucket.oss-cn-hangzhou.aliyuncs.com
    bucket: maimidoudou6firstbucket
    secure: false
```

保存退出后重启整个集群

```
docker-compose down
docker-compose up
```
