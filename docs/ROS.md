# 运行 ROS
安装完 docker 后，就可以运行 ROS 了。ROS 的 docker 镜像在 [Docker Hub](https://hub.docker.com/_/ros) 上可以找到，如果要运行 Desktop 版本的 ROS：[OSRF's Docker Hub](https://hub.docker.com/r/osrf/ros)。所有 ROS1 和 ROS2 的 docker 镜像都可以在上面两个链接找到（在 Tags 下搜索即可），各个版本的区别（如 core 和 desktop）见：[ROS1](https://ros.org/reps/rep-0150.html) 和 [ROS2](https://ros.org/reps/rep-2001.html)。下面以运行目前项目最常用的 ROS Noetic 为例，其他版本的 ROS 也类似。

1. 拉取镜像
```bash
docker pull osrf/ros:noetic-desktop-full
```
耐心等候下载完成即可。下载完成后，可以使用命令查看本地的镜像列表：
```bash
docker images
```
或
```bash
docker image ls
```
如果输出类似下面的内容，说明下载成功：
```
REPOSITORY    TAG                   IMAGE ID       CREATED        SIZE
osrf/ros      noetic-desktop-full   1f88719eb495   3 weeks ago    4.55GB
hello-world   latest                74cc54e27dc4   3 months ago   10.1kB
```
2. 运行容器
```bash
docker run -it \
    -e DISPLAY \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    --name noetic \
    -h $HOSTNAME \
    -w /root \
    --privileged \
    osrf/ros:noetic-desktop-full \
    bash
```
解释：
- `-it`: 交互式终端
- `-e DISPLAY`: 将主机的 DISPLAY 环境变量传递给容器
- `-v /tmp/.X11-unix:/tmp/.X11-unix`: 将主机的 X11 套接字挂载到容器中，以便容器可以访问主机的显示器
> GUI 支持见：[运行 GUI 应用](GUI.md)
- `--name noetic`: 给容器起个名字，方便后续操作
- `-h $HOSTNAME`: 将容器的主机名设置为主机的主机名
- `-w /root`: 将容器的工作目录设置为 `/root`
- `--privileged`: 以特权模式运行容器，允许容器访问主机的所有设备，以便在容器中也可以挂载 U 盘等设备。另一个典型例子是允许执行`sysctl -p`命令，修改内核参数。
- `osrf/ros:noetic-desktop-full`: 使用的镜像
- `bash`: 运行的命令

其他常用参数（详见[docker run cli](https://docs.docker.com/reference/cli/docker/container/run)）：
- `-d`: 后台运行容器
- `--rm`: 容器停止后自动删除容器
- `--network host`: 使用主机的网络，以便容器可以访问主机的网络，方便容器直接访问连接到主机的设备
- `-p <host_port>:<container_port>`: 将主机的端口映射到容器的端口
> ⚠️ **注意**：如果使用了`--network host`参数，则不能使用`-p`参数，因为容器会直接使用主机的网络。如果使用了`-p`参数，则不能使用`--network host`参数，因为容器会使用 NAT 网络。
- `-v <host_path>:<container_path>`: 将主机的目录挂载到容器中（主机上目录不存在时会自动创建）
- `--mount type=bind,source=<host_path>,target=<container_path>`: 将主机的目录挂载到容器中（主机上目录不存在时会报错）

⚠️ **强烈建议**：创建容器时挂载一个工作目录到容器中，方便主机和容器之间共享数据。可以使用如下命令创建一个名为`ros_ws`的工作目录，并将其挂载到容器的 `/root/ros_ws` 目录下：
```bash
cd && mkdir ros_ws
docker run -it \
    -e DISPLAY \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    --name noetic \
    -h $HOSTNAME \
    -w /root \
    --privileged \
    --mount type=bind,source=./ros_ws,target=/root/ros_ws \
    osrf/ros:noetic-desktop-full \
    bash
```
`-v`和`--mount`参数支持使用主机上的相对路径，但前面必须加上`./`或`../`之类，否则会报错（如上面的命令改成`source=ros_ws`）。也可以使用绝对路径，如`/home/user/ros_ws`或`$HOME/ros_ws`。目前还不支持使用`~`符号来表示主目录。
> ⚠️ **注意**：谨慎挂载系统目录，任何对挂载目录的修改都会直接影响到主机上的目录，可能会导致系统崩溃或数据丢失，建议挂载工作目录或数据目录。同时，如果创建容器前`<container_path>`已经存在，则会覆盖容器内的目录。
- `--gpus all`: 使用所有 GPU
> ⚠️ **注意**：如果使用了`--gpus`参数，则需要安装 NVIDIA Container Toolkit，见 [GPU 支持](GPU.md)。
3. （可选）运行成功后，容器内的命令行提示符会变成`root@<hostname>:~#`，说明已经进入了容器，可以使用`roscore`命令启动 ROS 核心。执行如下命令来安装镜像最小化时被删除的包，以恢复一个完整的 Ubuntu 系统：
```bash
unminimize
```
4. 运行完毕后，可以使用`exit`命令或`Ctrl+D`退出容器。
5. 其它命令：
- 重新进入容器：
```bash
docker start -ai noetic
```
这个命令等同于：
```bash
docker start noetic && docker attach noetic
```
前者启动容器，后者附加到容器的终端。注意：如果容器已经停止，则需要使用`docker start noetic`命令先启动容器，然后再使用`docker attach noetic`命令附加到容器的终端。

如果在多个终端中运行`docker attach`命令，在一个终端中运行命令，你会发现在其他终端中也会看到相同的输出，这是因为`docker attach`命令会将容器的标准输入输出流附加到当前终端中。因此，如果在一个终端中运行`exit`，就会导致容器停止运行，其他终端也会退出。为了避免这个问题，可以使用`docker exec`命令在容器中开启一个新的终端，这样在一个终端中运行`exit`，其他终端不会受到影响。
- 如果要开启一个新的终端窗口，可以使用`docker exec`命令：
```bash
docker exec -it noetic bash
```
这个命令会在容器中开启一个新的 bash 终端，类似于在容器中运行`bash`命令，执行`exit`后容器也不会停止运行。这样做的好处是可以在一个终端中运行命令，在另一个终端中查看输出，而不会影响容器的运行。同时，当多人使用同一个容器时（例如 SSH 连入同一个容器），也可以使用`docker exec`命令在容器中开启一个新的终端，就不会影响其他人的操作。
- 停止容器：
```bash
docker stop noetic
```
- 删除容器：
```bash
docker rm noetic
```
如果容器仍在运行中，则需要先停止容器，然后再删除容器，或者使用`-f`参数强制删除：
```bash
docker rm -f noetic
```
- 查看所有容器的状态：
```bash
docker ps -a
```
- 删除镜像：
```bash
docker rmi osrf/ros:noetic-desktop-full
```
如果当前镜像被容器使用，则需要先删除容器，然后再删除镜像，或者使用`-f`参数强制删除：
```bash
docker rmi -f osrf/ros:noetic-desktop-full
```
- 从主机拷贝文件到容器：
```bash
docker cp <host_path> <container_id>:<container_path>
```
> ⚠️ **注意**：虽然按照[官方文档](https://docs.docker.com/reference/cli/docker/container/cp)，此命令相当于`cp -a`，不会拷贝文件所有者信息，只会拷贝权限，但亲测并非如此，因此拷贝 SSH 公钥到容器后，文件所有者若还是主机用户而非 root，会导致 SSH fallback 到密码登录，请自行使用`chown`命令修改文件所有者为 root。
- 从容器拷贝文件到主机：
```bash
docker cp <container_id>:<container_path> <host_path>
```
