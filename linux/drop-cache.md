### Drop Cache and FS Cache in Linux

```bash
echo 0 > /sys/module/zfs/parameters/zfs_arc_shrinker_limit
echo 3 > /proc/sys/vm/drop_caches
```