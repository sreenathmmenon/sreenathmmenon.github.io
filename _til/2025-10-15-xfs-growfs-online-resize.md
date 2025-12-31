---
title: "xfs_growfs for Online Filesystem Expansion (No Downtime)"
date: 2025-10-15
tags: [linux, xfs, lvm, storage]
---

After the MongoDB disk-full incident, I decided to permanently expand `/var/log` instead of just truncating logs.

The server used XFS filesystem on LVM. Discovered I could expand it while mounted - no downtime needed!

```bash
# Check filesystem type
df -T /var/log
Filesystem                    Type  Size  Used
/dev/mapper/rootvg-var_loglv  xfs   2.0G  1.8G

# Check available space in volume group
vgs
VG     VSize   VFree
rootvg 100.00g 10.00g  # Have 10G free

# Extend the logical volume
lvextend -L +2G /dev/mapper/rootvg-var_loglv

# Grow the XFS filesystem (WHILE MOUNTED!)
xfs_growfs /var/log

# Verify
df -h /var/log
Filesystem                     Size  Used
/dev/mapper/rootvg-var_loglv  4.0G  1.8G
```

**Key points:**
- `xfs_growfs` works on mounted filesystems (no unmount needed)
- For ext4, use `resize2fs` instead
- Always check `vgs` first to ensure free space exists
- The command: `xfs_growfs /mount/point` (NOT the device path)

Expanded the filesystem during business hours, zero downtime. MongoDB kept running the whole time.

**For ext4 filesystems, same process but different resize command:**
```bash
lvextend -L +2G /dev/mapper/rootvg-var_loglv
resize2fs /dev/mapper/rootvg-var_loglv
```
