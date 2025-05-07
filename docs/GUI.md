# 运行 GUI 应用
运行 GUI 应用有两种方式：
- 挂载 X11 Socket（推荐，仅限 Docker Engine，不支持 Docker Desktop）
- 使用 SSH 转发 X11（支持 Docker Engine 和 Docker Desktop）
## 挂载 X11 Socket
这种方式的原理是将主机的 X11 Socket 挂载到容器中，运行容器时需要添加以下参数：
```bash
docker run -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY
```
部分时候主机上并没有`/tmp/.X11-unix`目录，此时需要使用`-v`参数而非`--mount`参数挂载。同时，需要在主机上运行以下命令：
```bash
xhost +local:root
# or
xhost +SI:localuser:root
# or
xhost +local:docker
# or
xhost + # Allow all users to access the X server, use with caution
```
这样就可以允许容器访问主机的显示器了。注意：这个命令会将主机的用户添加到 X11 的访问列表中，可能会导致安全隐患，所以建议在使用完毕后，运行以下命令将主机的用户从 X11 的访问列表中删除：
```bash
xhost -local:root
# or
xhost -SI:localuser:root
# or
xhost -local:docker
# or
xhost -
```
## 使用 SSH 转发 X11
这种方式的原理是使用 SSH 转发 X11，运行容器时需要添加以下参数：
```bash
docker run -p 127.0.0.1:<port>:22 # 注意这里必须用 127.0.0.1, 不支持 localhost 的语法
```
其中`<port>`是我们指定的端口号，例如`2222`。运行容器后，需要安装 SSH 服务：
```bash
apt update
apt install openssh-server
service ssh start # 注意 docker 默认不支持 systemd，所以不能使用`systemctl`命令
```
后续自行配置 SSH 公钥和私钥，或者使用密码登录。用密码登录时，记得修改`/etc/ssh/sshd_config`文件中的`PermitRootLogin`参数为`yes`，并重启 SSH 服务：
```bash
service ssh restart 
```
如果要想容器启动时自动启动 SSH 服务，可以在`~/.bashrc`中添加：
```bash
service ssh status > /dev/null 2>&1 || service ssh start
```
因为在创建容器时最后一个参数是`bash`，所以容器启动时`~/.bashrc`会被`source`，就实现了容器启动时自动启动 SSH 服务。使用这种方式时，需要在主机上运行：
```bash
ssh -X -p <port> root@localhost
```
`-X`参数表示使用 X11 转发，会自动在容器内设置`DISPLAY`环境变量；`-p`参数表示指定端口号。或者使用`~/.ssh/config`配置文件，添加以下内容：
```bash
Host docker
    HostName localhost
    Port <port>
    User root
    ForwardX11 yes
```
这样就可以使用`ssh docker`命令登录容器了。
## 验证生效
使用以上两个方法后，可以验证是否生效。首先安装`x11-apps`包：
```bash
apt install x11-apps
```
然后运行：
```bash
xclock
```
如果显示时钟，则说明 X11 转发生效。其他命令如`xeyes`、`xcalc`等也可以验证。
