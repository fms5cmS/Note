## 容器数据卷

卷就是目录或文件，存在于一个或多个容器中，由 Docker 挂载到容器。容器数据卷(Volume) 用于容器的持久化、容器间继承+共享数据。不属于联合文件系统，因此能够绕过 UnionFS 提供一些用于持续存储或共享数据的特性：**完全独立于容器的生存周期**，因此 Docker 不会在容器删除时删除其挂载的数据卷。

Volume 机制允许你将宿主机上指定的目录或文件挂载到容器里面进行读取和修改操作。

- 数据卷可在容器之间共享或重用数据
- 数据卷中的更改可以直接生效
- 数据卷中的更改不会包含在镜像的更新中
- 数据卷的生命周期一直持续到没有容器使用它为止

有两种方式向容器内添加数据卷：

- 方式一：通过命令 `docker run -v /宿主机绝对路径目录:/容器内目录 镜像名` 添加，如
  - `docker run -v /test ...` 没有显式声明宿主机目录，Docker 就会默认在宿主机上创建一个临时目录 /var/lib/docker/volumes/[VOLUME_ID]/_data，然后把它挂载到容器的 /test 目录上
  - `docker run -v /home:/test ...`，Docker 就直接把宿主机的 /home 目录挂载到容器的 /test 目录上
  - 使用 `docker inspect 容器id` 查看数据卷是否挂载成功
- 方式二：在 Dockerfile 文件中使用 `VOLUME` 指令来给镜像添加
  - 注意：出于可移植和分享的考虑，用 `-v 主机目录:容器目录` 这种方法不能够直接在Dockerfile中实现，因为宿主机目录是依赖于特定宿主机的，并不能够保证在所有的宿主机上都存在这样的特定目录。

如果 Docker 挂载主机目录后，Docker 访问出现 cannot open directory .: Permission denied

解决办法：在挂载目录后多加一个 `--privileged=true` 参数即可

```
docker create volume demo  # 创建一个数据卷
docker run -v demo:/tmp busybox:1.25 sh -c "date > /tmp/demo.log"
```

