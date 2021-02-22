# Requirements

Each worker node MUST have the following `sysctl` and `ulimit` changes made:

```bash
sysctl -w vm.max_map_count=262144
sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```
