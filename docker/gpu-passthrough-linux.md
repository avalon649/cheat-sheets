

```bash
apt-get update
apt-get upgrade

apt-get install qemu-guest-agent 
apt-get install build-essential 
```

```bash
apt install --no-install-recommends nvidia-cuda-toolkit nvidia-headless-530 nvidia-utils-530 libnvidia-encode-530
```

1. Reboot.

2. Then install nvtop

```bash
apt-get install nvtop
```

### Setting up NVIDIA Container Toolkit

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)

curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | apt-key add -

curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | tee /etc/apt/sources.list.d/nvidia-docker.list

apt-get update && apt-get install -y nvidia-container-toolkit

apt-get install nvidia-container-runtime

```
 
## update `daemon.json`

```bash
nano /etc/docker/daemon.json
```
## replace with

```bash
{
  "default-runtime": "nvidia",
  "runtimes": {
    "nvidia": {
      "path": "/usr/bin/nvidia-container-runtime",
      "runtimeArgs": []
    }
  }
}

```

```bash
apt-get install -y nvidia-docker2
```

```bash
systemctl restart docker
```

```bash
docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
```


