## Dockerfile 指令

### COPY复制文件

格式：

* `COPY [--chown=<user>:<group>] <源路径>... <目标路径>`
* `COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]`

和`RUN`指令一样，也有两种格式，一种类似于命令行，一种类似于函数调用。

举个栗子

``` shell
COPY package.json /usr/src/app/
COPY --chown=55:mygroup files* /mydir/
```

这里需要注意的是源路径表示build是传递的上下文，目标路径在不存在的情况下是可以自己创建的

### ADD更高级的复制文件

`ADD`指令在`COPY`的格式上添加了一些功能

1. 源路径可以指定为网络路径，下载之后权限自动设置为600。

不推荐该用法，因为下载后涉及到权限的修改，如果是压缩文件还需要去解压，不如直接使用RUN命令方便点，也可以防止生产的临时文件没有及时清理

2. 自动解压缩

``` shell
FROM scratch
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /
...
```

压缩格式为gzip,bzip2以及xz的情况下，`ADD`指令将会自动进行解压缩操作。

这是条指令唯一推荐使用的场景。

同样的可以添加改变文件所述用户及用户组的命令。

``` shell
ADD --chown=55:mygroup files* /mydir/
ADD --chown=bin files* /mydir/
```

### CMD容器启动命令

格式：

* shell格式：CMD<命令>
* exec格式：CMD ["可执行文件", "参数1", "参数2"...] #**推荐**
* 参数列表格式：`CMD ["参数1", "参数2"...]`。在指定了 `ENTRYPOINT` 指令后，用 `CMD` 指定具体的参数。

参考文档

- `Dockerfie` 官方文档：<https://docs.docker.com/engine/reference/builder/>
- `Dockerfile` 最佳实践文档：<https://docs.docker.com/develop/develop-images/dockerfile_best-practices/>
- `Docker` 官方镜像 `Dockerfile`：<https://github.com/docker-library/docs>