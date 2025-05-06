# 运行 GUI 应用
运行 GUI 应用有两种方式：
- 挂载 X11 Socket
- 使用 SSH 转发 X11
## 挂载 X11 Socket
这种方式是将主机的 X11 Socket 挂载到容器中，这样容器就可以直接访问主机的显示器了。使用这种方式时，需要在运行`docker run`时添加以下参数：
```bash
-v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY 
```
这样容器就可以直接访问主机的显示器了。使用这种方式时，需要在主机上运行以下命令：
```bash
xhost +local:root
# or
xhost +SI:localuser:root
# or
xhost +local:docker
# or
xhost + # Allow all users to access the X server, use with caution
```
这样就可以允许容器访问主机的显示器了。注意：这个命令会将主机的所有用户都添加到 X11 的访问列表中，这样会导致安全隐患，所以建议在使用完毕后，运行以下命令将主机的所有用户都删除：
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
这种方式是使用 SSH 转发 X11，这样容器就可以通过 SSH 访问主机的显示器了。使用这种方式时，需要在运行`docker run`时添加以下参数：
```bash
-p 127.0.0.1:<port>:22 # 注意这里必须用 127.0.0.1 , -p 不支持 localhost 的语法
```
其中`<port>`是我们指定的端口号，例如`2222`。运行容器后，需要安装 SSH 服务：
```bash
apt update
apt install openssh-server
service ssh start
```
后续自行配置 SSH 公钥和私钥，或者使用密码登录。用密码登录时，记得修改`/etc/ssh/sshd_config`文件中的`PermitRootLogin`参数为`yes`，并重启 SSH 服务：
```bash
service ssh restart # 注意 docker 默认不支持 systemd，所以不能使用 systemctl 命令
```
如果要想容器启动时自动启动 SSH 服务，可以在`.bashrc`中添加：
```bash
service ssh status > /dev/null 2>&1 || service ssh start
```
这样容器就可以通过 SSH 访问主机的显示器了。使用这种方式时，需要在主机上运行以下命令：
```bash
ssh -X -p <port> root@localhost
```
或者使用`ssh_config`配置文件，添加以下内容：
```bash
Host docker
    HostName localhost
    Port <port>
    User root
    ForwardX11 yes
```
这样就可以使用`ssh docker`命令登录容器了。
