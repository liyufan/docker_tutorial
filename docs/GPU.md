## GPU 支持
见 [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html). 验证生效：
```bash
docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```
