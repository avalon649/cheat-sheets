### List all Docker images on k3s

```bash
k3s crictl images
```

### Delete Docker images that are not used

```bash
k3s crictl rmi --prune
```