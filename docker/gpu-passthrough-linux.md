

```bash
sudo apt-get update

sudo apt-get upgrade

sudo apt-get install qemu-guest-agent 

sudo apt-get install build-essential 
```

```bash
sudo apt install --no-install-recommends nvidia-cuda-toolkit nvidia-headless-470 nvidia-utils-470 libnvidia-encode-470
```

1. Reboot.

2. Then install nvtop

```bash
sudo apt-get install nvtop
```

### Setting up NVIDIA Container Toolkit

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)

curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -

curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit

sudo apt-get install nvidia-container-runtime

```
 
## update `daemon.json`

```bash
sudo nano /etc/docker/daemon.json
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
sudo apt-get install -y nvidia-docker2
```

```bash
sudo systemctl restart docker
```

```bash
sudo docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
```


