---
title: "Extending XFS Filesystem on LVM When Logs Fill Up"
date: 2025-08-15
tags: [linux, xfs, lvm, rhel, storage]
---

MongoDB crashed because `/var/log` hit 100% capacity. Instead of just truncating logs (temporary fix), I extended the filesystem permanently.

```bash
# Check filesystem type
df -T /var/log
Filesystem                    Type  Size  Used Avail Use% Mounted on
/dev/mapper/rootvg-var_loglv  xfs   2.0G  2.0G   20K 100% /var/log

# Check available space in volume group
vgs
VG     #PV #LV #SN Attr   VSize   VFree
rootvg   1  10   0 wz--n- 100.00g 10.00g

# Extend logical volume by 2GB
lvextend -L +2G /dev/mapper/rootvg-var_loglv

# Grow XFS filesystem (online, no downtime!)
xfs_growfs /var/log

# Verify the expansion
df -h /var/log
Filesystem                     Size  Used Avail Use% Mounted on
/dev/mapper/rootvg-var_loglv  4.0G  1.3G  2.8G  32% /var/log
```

**Key points:**
- `xfs_growfs` works on mounted filesystem (no downtime)
- For ext4, use `resize2fs` instead
- Always check VG free space with `vgs` before extending

**Pro tip:** Use `lsblk -f` to see all filesystems and their types at once.