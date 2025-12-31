---
title: "df -h vs du -sh: When to Use Which"
date: 2025-11-12
tags: [linux, disk-space, troubleshooting]
---

Was debugging disk space issues on a server. `/var/log` showed full in `df`, but `du` showed different numbers. Confused me until I understood the difference.

**df (disk free)** - Shows filesystem/partition usage:

```bash
df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/rootvg-var_loglv  2.0G  2.0G   20K 100% /var/log
/dev/mapper/rootvg-rootlv      50G   35G   15G  70% /
```

Shows partition-level stats. Good for: "Is my partition full?"

**du (disk usage)** - Shows file/directory sizes:

```bash
du -sh /var/log/*
1.2G  /var/log/mongodb
200M  /var/log/nginx
150M  /var/log/syslog
# Total: 1.55G
```

Shows actual file sizes. Good for: "What's taking up space?"

**Why they differ:**

```bash
# df shows 2.0G used
# du shows 1.55G used
# Difference: deleted files still held open by processes!
```

Deleted files keep using space until the process closes them. `df` counts them, `du` doesn't.

**My workflow:**
1. `df -h` → See which partition is full
2. `du -sh /path/*` → Find what's using space
3. If they don't match → Look for deleted-but-open files: `lsof | grep deleted`
