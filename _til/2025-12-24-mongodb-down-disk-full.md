---
title: "MongoDB Crashed Due to Full /var/log Partition"
date: 2025-12-24
tags: [mongodb, linux, troubleshooting, rhel]
---

Our project UI stopped functioning. Investigation showed MongoDB was down on the RHEL VM. Root cause: `/var/log` partition was 100% full.

```bash
# Check disk usage
df -h
/dev/mapper/rootvg-var_loglv  2.0G  2.0G  20K  100% /var/log

# Find large log files
du -sh /var/log/* | sort -rh | head -5

# Truncate log file without deleting (keeps file handles intact)
truncate -s 0 /var/log/mongodb/mongod.log

# Or for multiple files
for log in /var/log/mongodb/*.log; do
  truncate -s 0 "$log"
done
```

**Why truncate instead of rm:**
- `rm` on active log file doesn't free space until process restarts
- `truncate -s 0` immediately frees space while keeping file handles valid
- MongoDB continues writing to same file descriptor

**Prevention:** Set up logrotate for MongoDB logs or increase `/var/log` partition size.