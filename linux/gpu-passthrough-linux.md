## Proxmox

Shut down your VM in proxmox, edit your conf file, it should be here (note, change path to your VM's ID)

`/etc/pve/qemu-server/100.conf`

add `cpu: host,hidden=1,flags=+pcid` to that file

start the server.

## Linux Guest

```bash
sudo apt-get update

sudo apt-get upgrade

sudo apt-get install qemu-guest-agent # this is optional if you are virtualizing this machine

sudo apt-get install build-essential # build-essential is required for nvidia drivers to compile

apt install linux-headers-$(uname -r) -y
apt install nvidia-headless-470-server nvidia-utils-470-server libnvidia-encode-470-server -y
```

Then reboot.

Then install `nvtop`

```bash
sudo apt-get install nvtop
```

## tensorflow workload

```bash
nvidia-docker run --rm -ti tensorflow/tensorflow:r0.9-devel-gpu
```

## Rancher / Kubernetes

In your Rancher server (or kubernetes host)

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)

curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -

curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit

sudo apt-get install nvidia-container-runtime
```

update `daemon.json`

```bash
sudo nano /etc/docker/daemon.json
```

Replace with:

```json
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

Install one more util for nvidia:

```bash
sudo apt-get install -y nvidia-docker2
```

Reboot

Then, using `kubectl` on your kubernetes / rancher host

```bash
kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/master/nvidia-device-plugin.yml
```