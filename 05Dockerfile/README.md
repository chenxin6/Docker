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

VOLUME 用于在 image 中创建一个挂载点目录，后面只能跟参数容器中的目录，所以它属于存储卷类型中的 `docker 管理的卷`，如果容器中挂载点目录路径下此前存在文件，则 docker run 会在挂载完后，将这些存在的文件都复制进 docker 管理的卷中

EXPOSE 用于为容器打开指定要监听的端口（只能绑定到宿主机的动态端口）以实现与外部通信，例如`EXPOSE 80/tcp`通常与 docker run 中的 -P 这个参数配合（注意大写），这个时候才是完成了暴露，Dockerfile 中只是说明可以暴露

ENV 为镜像定义所需的环境变量，例如`ENV PATH="/opt/gtk/bin:${PATH}"`

RUN 构建镜像时要运行的命令，可以通过 && 将多个命令连起来从而减少镜像层数，它有两种格式：

- `RUN <command>`
    - `<command>`通常是一个 shell 命令，且以`/bin/sh -c`来运行它，这就意味着此进程在容器中的 PID 不为1，不能接收 Unix 信号，因此当使用 docker stop 命令停止容器时，此进程接收不到 SIGTERM 信号
    - 这里简述下 docker stop 的原理，它向容器中 PID 为1的进程发送 SIGTERM 信号，并给予10秒钟（可用参数`--time`）清理，超时才`-9`强杀，这样就可以比较优雅的关闭容器。`/bin/sh -c`是一个 PID 为1的进程，它收到了 SIGTERM 却不会转发给它的子命令，这样就造成了 `/bin/sh -c` 收到 SIGTERM 未作响应被强杀，同时把它的子进程毫无征兆的干掉了。例如在 Java 中用`Runtime.addShutdownHook()`是捕获不到该信号的，从而不能触发相应的钩子函数
- `RUN ["<executable>", "<param1>", "<param2>"]`
    - 该格式的参数是一个 JSON 格式的数组，其中`<executable>`为要运行的命令，后面的`<paramN>`为传递给命令的选项或参数，然而此种格式指定的命令不会以`/bin/sh -c`来运行它，因此常见的 shell 操作如环境变量替换以及通配符替换（*、?等）都将不会进行，不过如果硬是想要运行命令依赖于此 shell 的特性，可以将其替换为类似下面的格式`RUN ["/bin/bash", "-c", "<executable>", "<param1>", "<param2>"]`

CMD 类似于 RUN 指令但 CMD 是指启动为容器时所要运行的命令，该命令运行结束后容器也将终止，它可以被 docker run 的命令中的选项所覆盖。CMD 有三种格式，前两种同 RUN，第三种是`CMD ["<param1>", "<param2>"]`用于为 ENTRYPOINT 指令提供默认参数

