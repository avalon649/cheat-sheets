# Disk Usage in Linux

| Command | Description |
| --- | --- |
| `du -x / \| sort -k1n \| tail -70` | Show largest directories on the root filesystem |
| `du -h --maxdepth=1 /` | Show largest directories on the root filesystem alternative |