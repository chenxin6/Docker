# 05Dockerfile

需求：直接下载的镜像并不完全符合生产环境的需求，例如需要更改配置文件等内容

解决方案：

- 自建镜像（麻烦）
- 通过环境变量的传入（老土）
- 通过 Dockerfile 结合阿里云容器镜像服务的自动构建生成镜像

**注意：Dockerfile 中的一条指令就会生成一层镜像，所以指令要尽量的少**

## 常用的指令关键字

FROM 表示基准镜像，第一条命令必须是 FROM，例如`FROM busybox:latest`

MAINTAINER 表示作者信息，例如`MAINTAINER:"chenxin <494954207@qq.com>"`，也可以使用 LABEL 表示，例如`LABEL maintainer:"chenxin <494954207@qq.com>"`

COPY 用于从宿主机的文件打包进创建的新映像文件，这里有几个注意点：

- src 必须是 build 上下文中的路径，不能是其父目录中的文件
- 如果是 src 目录，则其内部文件或子目录会被递归复制，但 src 目录自身不会被复制
- 如果指定了多个 src 或在 src 中使用了通配符，则 dest 必须是一个目录且必须以 / 结尾
- 如果 dest 事先不存在，它将会被自动创建，这也包括其父目录路径

ADD 指令和 COPY 类似，并且 ADD 支持使用 tar 文件和 url 路径

WORKDIR 改变工作目录，允许多次切换

USER 用于指定运行镜像时或运行 Dockerfile 中任何 RUN、CMD 或 ENTRYPOINT 指令指定的程序时的用户名或 UID，格式为`USER <UID>|<UserName>`，需要注意的是`<UID>`可以为任意的数字，但实践中其必须为`/etc/passwd`中某用户的有效 UID，否则`docker run`命令将运行失败

VOLUME 用于在 image 中创建一个挂载点目录，后面只能跟参数容器中的目录，所以它属于存储卷类型中的 `docker 管理的卷`，如果容器中挂载点目录路径下此前存在文件，则 docker run 会在挂载完后，将这些存在的文件都复制进 docker 管理的卷中

EXPOSE 用于为容器打开指定要监听的端口（只能绑定到宿主机的动态端口）以实现与外部通信，例如`EXPOSE 80/tcp`通常与 docker run 中的 -P 这个参数配合（注意大写），这个时候才是完成了暴露，Dockerfile 中只是说明可以暴露

ENV 为镜像定义所需的环境变量，例如`ENV PATH="/opt/gtk/bin:${PATH}"`

RUN 构建镜像时要运行的命令，可以通过 && 将多个命令连起来从而减少镜像层数，它有两种格式：

- `RUN <command>`
    - 针对启动命令 CMD 的情况：`<command>`通常是一个 shell 命令，默认`/bin/sh -c`来运行它，但是这个 shell 命令的 PID 为1，因为在这种格式下会将原来 PID 为1的进程（也就是 shell）替换成这个 shell 命令，这就意味着该 shell 命令能接收 Unix 信号，因此当使用 docker stop 命令停止容器时，此进程能接收到 SIGTERM 信号
- `RUN ["<executable>", "<param1>", "<param2>"]`
    - 针对启动命令 CMD 的情况：该格式的参数是一个 JSON 格式的数组，其中`<executable>`为要运行的命令，后面的`<paramN>`为传递给命令的选项或参数，然而此种格式指定的命令不会以`/bin/sh -c`来运行它，因此常见的 shell 操作如环境变量替换以及通配符替换（*、?等）都将不会进行，不过如果硬是想要运行命令依赖于此 shell 的特性，可以将其替换为类似下面的格式`RUN ["/bin/bash", "-c", "<executable>", "<param1>", "<param2>"]`
    - 这里简述下 docker stop 的原理，它向容器中 PID 为1的进程发送 SIGTERM 信号，并给予10秒钟（可用参数`--time`）清理，超时才`-9`强杀，这样就可以比较优雅的关闭容器。`/bin/sh -c`是一个 PID 为1的进程，它收到了 SIGTERM 却不会转发给它的子命令，这样就造成了`/bin/sh -c`收到 SIGTERM 未作响应被强杀，同时把它的子进程毫无征兆的干掉了。例如在 Java 中用`Runtime.addShutdownHook()`是捕获不到该信号的，从而不能触发相应的钩子函数

CMD 类似于 RUN 指令但 CMD 是指启动为容器时所要运行的命令，该命令运行结束后容器也将终止，它可以被 docker run 的命令中的选项所覆盖。CMD 有三种格式，前两种同 RUN，第三种是`CMD ["<param1>", "<param2>"]`用于为 ENTRYPOINT 指令提供默认参数，即如果有 ENTRYPOINT 指令的话就会把 CMD 后面的内容（数组的格式）作为参数给 ENTRYPOINT

ENTRYPOINT `ENTRYPOINT /bin/httpd -f -h ${WEB_DOC_ROOT}`同上默认是使用`/bin/sh -c`来运行它，所以不要有例1这种情况，例1和例2两种写法都会因为主进程结束而导致容器被关闭，且这两种写法中运用到了 CMD 的第三种写法，所以 CMD 中的内容还是会被`docker run`启动容器时的指定内容所覆盖

- 例1，没有语法错误但是有冗余
    ```
    CMD ["/bin/httpd", "-f", "-h ${WEB_DOC_ROOT}"]
    ENTRYPOINT /bin/sh -c
    ```
- 例2，正确的写法
    ```
    CMD ["/bin/httpd", "-f", "-h ${WEB_DOC_ROOT}"]
    ENTRYPOINT ["/bin/sh", "-c"]
    ```

HEALTHCHECK 健康检查

- `HEALTHCHECK NONE` 拒绝任何健康检查，包括 from 引入的 base 中的所有健康检查
- `HEALTHCHECK --start-period=3s --interval=5s --timeout=3s CMD wget -O - -q http://${IP:-0.0.0.0}:${PORT:-80} || exit 1` 初始化时间为3秒，也就是说在开始的3秒钟内不会进行健康检查，每隔5秒检查一次，如果超时时间超过3秒则退出表示不健康（状态码为1，0是正常）

## ENTRYPOINT 的高级用法

以开启 nginx 服务为例，我们在编写 Dockerfile 的基础上可以再编写一个 entrypoint.sh，具体内容详见文件夹 nginx。值得注意的是 entrypoint.sh 使用双引号而不是单引号，且其中由于 CMD 的内容是 ENTRYPOINT 的参数内容，所以我们要在 entrypoint.sh 的最后一行使用`exec "$@"`将这些传入的参数执行，这样就会保证 CMD 所运行内容的 PID 为1

## PID 进程号的思考

容器启动后会执行一个 PID 为1的进程，所以对于`CMD /bin/sh`来说，容器中 PID 为1的进程是 shell，如果我们改成`CMD st.cmd`那么情况是先启动 PID 为1的进程也就是 shell，然后shell 作为父进程创建脚本`st.cmd`子进程，然后再让`st.cmd`的 PID 为1

对于`CMD ["ls", "/"]`的情况是直接通过内核运行`ls \`所以 PID 为1的不是 shell 而是`ls \`，正因为它不是通过 shell 运行的所以它不能使用 shell 的一些特性，比如`CMD ["echo" "$EPICS_BASE"]`是错误的语法

## `CMD st.cmd`和`CMD ["/bin/sh", "-c", "st.cmd"]`的区别（st.cmd 是 EPICS 的启动脚本）

前者容器可以正常运行，并且能够通过`docker attach`进行交互，后者由于 PID 为的1的进程是 shell 所以这个主进程会在运行完 st.cmd 后去运行下一个命令，但是由于没有下一个命令了所以会导致主进程结束，这就会导致容器被关闭

# Demo 演示
## 目录结构
```
TOP
└───Dockerfile          构建镜像的文件
```
## Dockerfile 的内容

```
FROM registry.cn-hangzhou.aliyuncs.com/maimidoudou6/mybusybox
LABEL maintainer="maimidoudou6 <maimidoudou6@gmail.com>" app="httpd"

ENV WEB_DOC_ROOT="/data/web/html/"
RUN echo '<h1>Dockerfile hello world</h1>' > ${WEB_DOC_ROOT}index.html

#CMD /bin/httpd -f -h ${WEB_DOC_ROOT}
#CMD ["/bin/sh", "-c", "/bin/httpd -f -h ${WEB_DOC_ROOT}"]
ENTRYPOINT /bin/httpd -f -h ${WEB_DOC_ROOT}
```
## 构建

在 TOP 目录底下运行该命令构建`docker build -t mybusybox:latest ./`

## 创建容器并检查是否正常

`docker run --name t1 --rm -p 10080:80 mybusybox`创建容器后`curl 127.0.0.1:10080`
