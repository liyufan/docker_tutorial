# 安装 docker
⚠️ **注意**：根据 docker 官方文档，只有 Windows 上的 Docker Desktop 支持调用显卡，Linux 上的 Docker Desktop 则不支持，见：[GPU support](https://docs.docker.com/desktop/features/gpu). 同时，由于 Linux 上的 Docker Desktop 本质是虚拟机，性能较差，且运行 GUI 应用只能透过 SSH 转发 X11，不支持挂载 X11 套接字的方法；这样以来，无法在 VSCode 中使用 Dev Containers 扩展直接连接，必须通过 SSH 连接容器，且需要配置容器的 SSH 开机自启动，极为繁琐，故本仓库只介绍 Docker Engine 的安装和使用。
## 安装 Docker Engine
官方教程： [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository). 安装完毕后，跳过第3步的`sudo docker run hello-world`，直接执行 post-installation，见 [Linux post-installation steps for Docker Engine](https://docs.docker.com/engine/install/linux-postinstall). 执行完`sudo usermod -aG docker $USER`后，请重启计算机，而不是按照提示注销当前用户，否则可能不生效。重启完计算机后，为避免多次运行生成多个容器，运行`hello-world`时请加上`--rm`参数:
```bash
docker run --rm hello-world
```
