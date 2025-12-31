---
title: "truncate -s 0 vs rm for Active Log Files"
date: 2025-12-18
tags: [linux, logging, disk-space]
---

MongoDB crashed. `/var/log` was 100% full. Deleted the huge log file with `rm`, but disk space didn't free up. MongoDB was still holding the file open.

**The problem with `rm` on active files:**

```bash
rm /var/log/mongodb/mongod.log
# File deleted from directory listing
# But MongoDB process still has file handle open
# Disk space NOT freed until MongoDB restarts
```

**The solution: `truncate -s 0`**

```bash
truncate -s 0 /var/log/mongodb/mongod.log
# File still exists
# Process keeps writing to same file handle
# Disk space freed IMMEDIATELY
```

Why it works:
- `rm` removes directory entry but file data remains while open
- `truncate -s 0` empties the file content while keeping it in place
- Process doesn't know/care - continues writing to same file descriptor

**For multiple log files:**

```bash
for log in /var/log/mongodb/*.log; do
  truncate -s 0 "$log"
done

# Instantly freed 1.8GB
```

**Better long-term solution:**
Set up logrotate or increase the `/var/log` partition size. But `truncate` is perfect for emergency disk space recovery when you can't restart the process.
