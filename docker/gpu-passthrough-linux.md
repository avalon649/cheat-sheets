

```bash
apt-get update
apt-get upgrade

sudo apt-get install qemu-guest-agent 

sudo apt-get install build-essential 
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
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update

sudo apt-get install -y nvidia-container-toolkit
```

### Install Nvidia Docker 2

```bash
apt-get install -y nvidia-docker2
```

### Update `daemon.json`

```bash
nano /etc/docker/daemon.json
```

### And Replace `daemon.json` With

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

### Restart Docker

```bash
systemctl restart docker
```

### Run A Sample Workload

```bash
docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
```


