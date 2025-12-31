---
title: "When Your Database Goes Down at 3 AM: A MongoDB War Story"
date: 2025-12-28
excerpt: "How a full /var/log partition crashed MongoDB in production, what I learned about truncate vs rm, and the permanent fix using LVM."
tags: [mongodb, linux, troubleshooting, devops, incident-response, lvm, xfs]
keywords: "MongoDB crash, disk full, truncate vs rm, LVM resize, XFS filesystem, production incident, log management"
---

3:17 AM. Phone buzzes. Monitoring alert: "Project UI is down. MongoDB connection failed."

I'm up, laptop open, VPN connecting. This is production. Customers are affected.

## The Initial Investigation

SSH into the server. Check MongoDB status:

```bash
systemctl status mongod

● mongod.service - MongoDB Database Server
   Active: inactive (dead)
   ...
   Dec 28 03:14:23 server systemd[1]: mongod.service: Failed
```

MongoDB is dead. Check the logs:

```bash
tail -100 /var/log/mongodb/mongod.log

# Nothing. File exists but tail shows nothing?

ls -lh /var/log/mongodb/mongod.log
-rw-r--r-- 1 mongodb mongodb 0 Dec 28 03:14 mongod.log

# File is empty. That's weird.
```

Try to start MongoDB:

```bash
systemctl start mongod

# Fails immediately

journalctl -u mongod -n 50
Dec 28 03:17:45 mongod[12453]: Error opening log file: No space left on device
```

There it is. **No space left on device.**

## Finding the Problem

Check disk space:

```bash
df -h

Filesystem                     Size  Used Avail Use% Mounted on
/dev/mapper/rootvg-rootlv       50G   35G   15G  70% /
/dev/mapper/rootvg-var_loglv   2.0G  2.0G   20K 100% /var/log  ← Here!
/dev/mapper/rootvg-homelv       20G   12G  8.0G  60% /home
```

`/var/log` is 100% full. MongoDB can't write logs, so it won't start.

What's taking up space?

```bash
du -sh /var/log/* | sort -rh | head -5

1.8G  /var/log/mongodb
150M  /var/log/nginx
 89M  /var/log/syslog
 45M  /var/log/kern.log
 32M  /var/log/audit
```

MongoDB logs: 1.8GB out of 2GB partition. That's the culprit.

```bash
ls -lh /var/log/mongodb/

-rw-r--r-- 1 mongodb mongodb 1.8G Dec 28 03:14 mongod.log.2024-12-27
-rw-r--r-- 1 mongodb mongodb    0 Dec 28 03:14 mongod.log
```

Yesterday's log file: 1.8GB. Today's: 0 bytes (couldn't be created - no space).

## First Attempt: rm (Wrong!)

My first instinct:

```bash
rm /var/log/mongodb/mongod.log.2024-12-27

# Check disk space
df -h /var/log
/dev/mapper/rootvg-var_loglv   2.0G  2.0G   20K 100% /var/log

# Still 100% full?!
```

The file is gone from the directory listing, but the space isn't freed. Why?

Checked for processes holding deleted files:

```bash
lsof | grep deleted | grep mongodb

# Nothing - MongoDB is stopped
```

Then I remembered: MongoDB was writing to that file when the partition filled. The file was still held open when MongoDB crashed. Even though MongoDB is stopped now, the file handle persisted.

**Lesson learned: `rm` on files that were open when a process crashed doesn't immediately free space.**

## The Quick Fix: truncate

Instead of deleting, empty the file:

```bash
truncate -s 0 /var/log/mongodb/mongod.log.2024-12-27

df -h /var/log
/dev/mapper/rootvg-var_loglv   2.0G  200M  1.8G  10% /var/log

# Space freed immediately!
```

**Why `truncate` works:**
- Empties file content while keeping the file in place
- File descriptor remains valid (even if process is dead)
- Disk space freed instantly
- Process can resume writing to same file

Start MongoDB:

```bash
systemctl start mongod
systemctl status mongod

● mongod.service - MongoDB Database Server
   Active: active (running)

# Success!
```

3:42 AM. MongoDB is back. UI is responsive. Crisis averted. For now.

## Root Cause Analysis

Why did the log grow so large? Checked MongoDB configuration:

```bash
cat /etc/mongod.conf

systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log
  logAppend: true
  # No log rotation configured!
```

No log rotation. MongoDB just kept appending to the same file forever. Eventually it filled the partition.

Combined with:
- `/var/log` partition: Only 2GB
- Production load: ~500MB logs per day
- No monitoring: Nobody noticed until it crashed

Perfect recipe for disaster.

## The Permanent Fix

Two options:

**Option 1:** Set up log rotation (temporary fix)
**Option 2:** Expand `/var/log` partition (permanent fix)

I chose both.

### Part 1: Expand the Partition

Check volume group:

```bash
vgs
VG     #PV #LV #SN Attr   VSize   VFree
rootvg   1  10   0 wz--n- 100.00g 12.00g

# 12GB free in volume group
```

Extend the logical volume:

```bash
# Add 4GB to /var/log (double its size)
lvextend -L +4G /dev/mapper/rootvg-var_loglv

Extending logical volume rootvg/var_loglv to 6.00 GiB
Logical volume rootvg/var_loglv successfully resized.
```

Grow the filesystem (XFS in this case):

```bash
# XFS can be grown while mounted - no downtime!
xfs_growfs /var/log

meta-data=/dev/mapper/rootvg-var_loglv
         =                       sectsz=512   attr=2, projid32bit=1
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 524288 to 1572864
```

Verify:

```bash
df -h /var/log
/dev/mapper/rootvg-var_loglv   6.0G  200M  5.8G   4% /var/log

# Now 6GB total!
```

**Key insight:** `xfs_growfs` works on mounted filesystems. No downtime needed. MongoDB kept running during the resize.

*Note: For ext4 filesystems, use `resize2fs` instead of `xfs_growfs`.*

### Part 2: Set Up Log Rotation

Created `/etc/logrotate.d/mongodb`:

```bash
/var/log/mongodb/*.log {
    daily
    rotate 7
    compress
    delaycompress
    notifempty
    create 0640 mongodb mongodb
    sharedscripts
    postrotate
        /bin/kill -SIGUSR1 `cat /var/run/mongodb/mongod.pid 2>/dev/null` 2> /dev/null || true
    endscript
}
```

This config:
- Rotates logs daily
- Keeps 7 days of logs
- Compresses old logs
- Signals MongoDB to reopen log file after rotation

Test it:

```bash
logrotate -f /etc/logrotate.d/mongodb

ls -lh /var/log/mongodb/
-rw-r----- 1 mongodb mongodb  89M Dec 28 04:15 mongod.log
-rw-r----- 1 mongodb mongodb 156M Dec 28 03:14 mongod.log.1
-rw-r----- 1 mongodb mongodb 234M Dec 27 23:59 mongod.log.2.gz
-rw-r----- 1 mongodb mongodb 189M Dec 26 23:59 mongod.log.3.gz
```

Perfect. Logs rotate daily, old ones compressed.

### Part 3: Monitoring

Added disk space monitoring:

```bash
# Check every 5 minutes
*/5 * * * * /usr/local/bin/check_disk_space.sh
```

`check_disk_space.sh`:

```bash
#!/bin/bash

THRESHOLD=80
USAGE=$(df -h /var/log | tail -1 | awk '{print $5}' | sed 's/%//')

if [ "$USAGE" -gt "$THRESHOLD" ]; then
    echo "/var/log is ${USAGE}% full" | mail -s "Disk Alert" ops@company.com
fi
```

Now we get alerts at 80% before it becomes critical.

## What I Learned

### 1. truncate -s 0 vs rm for Active Log Files

**Use `truncate` when:**
- Process is still running and writing to file
- Process crashed but file was open
- You need space back immediately
- Can't restart the process

**Use `rm` when:**
- File is not open by any process
- You want the file gone completely
- Normal log cleanup

**Check open files:**
```bash
lsof | grep deleted
```

If files show up as "deleted" but still using space, the process still has them open.

### 2. XFS vs ext4 Online Resize

**XFS:**
```bash
xfs_growfs /mount/point  # Can only grow (not shrink)
```

**ext4:**
```bash
resize2fs /dev/mapper/device  # Can grow or shrink
```

Both can be done online (mounted). XFS is simpler for growing filesystems.

### 3. LVM is Your Friend

Without LVM, expanding a partition requires:
- Backing up data
- Repartitioning
- Restoring data
- Hours of downtime

With LVM:
```bash
lvextend -L +4G /dev/mapper/volume
xfs_growfs /mount/point
# Done in seconds, no downtime
```

### 4. Always Have Log Rotation

MongoDB doesn't rotate logs by default. Neither do many other applications. Always set up `logrotate`:

```bash
ls /etc/logrotate.d/
apache2  mongodb  nginx  postgresql  syslog  ...
```

Every service that writes logs should have a logrotate config.

### 5. Monitor Disk Space Proactively

Don't wait for failures. Monitor at multiple thresholds:

- 70% - Warning (nice to know)
- 80% - Alert (take action soon)
- 90% - Critical (immediate action required)
- 95% - Emergency (might fail any moment)

### 6. Understand df vs du

**df (disk free)** - Shows filesystem usage (includes open deleted files)
**du (disk usage)** - Shows actual file sizes (excludes deleted files)

If `df` shows more usage than `du`, you have deleted-but-open files.

### 7. Plan Partition Sizes Carefully

Our original 2GB `/var/log` was too small:

```bash
# Current daily log generation
MongoDB: 500MB/day
Nginx: 50MB/day
System: 100MB/day
Total: ~650MB/day

# With 7-day retention = 4.5GB needed
# Gave 6GB = comfortable headroom
```

General rule: (daily log size) × (retention days) × 1.5 for safety margin.

## Post-Incident Review

**Timeline:**
- 03:14 AM: Partition fills, MongoDB crashes
- 03:17 AM: Alert received
- 03:25 AM: Problem identified (disk full)
- 03:35 AM: Quick fix applied (truncate)
- 03:42 AM: MongoDB restored
- 04:30 AM: Permanent fix implemented (LVM extend + logrotate)
- 05:00 AM: Monitoring added

**Total downtime:** 28 minutes

**Impact:**
- UI unavailable for 28 minutes
- No data loss (MongoDB clean shutdown)
- ~50 users potentially affected (low traffic time)

**Preventive Measures Added:**
1. ✓ Expanded `/var/log` from 2GB to 6GB
2. ✓ Configured log rotation (7-day retention)
3. ✓ Added disk space monitoring
4. ✓ Documented `truncate` procedure
5. ✓ Created runbook for disk-full scenarios

## The Runbook

Created for next time:

```markdown
# MongoDB Down - Disk Full

## Symptoms
- MongoDB won't start
- Error: "No space left on device"
- `df -h` shows 100% on /var/log

## Quick Fix (5 min)
1. Identify full partition: `df -h`
2. Find large files: `du -sh /var/log/* | sort -rh`
3. Truncate old logs: `truncate -s 0 /var/log/mongodb/old.log`
4. Restart MongoDB: `systemctl restart mongod`

## Permanent Fix (30 min)
1. Check VG space: `vgs`
2. Extend LV: `lvextend -L +4G /dev/mapper/rootvg-var_loglv`
3. Grow filesystem: `xfs_growfs /var/log` (or `resize2fs` for ext4)
4. Set up logrotate: `/etc/logrotate.d/mongodb`
5. Test rotation: `logrotate -f /etc/logrotate.d/mongodb`

## Verify
- MongoDB running: `systemctl status mongod`
- Logs rotating: Check `/var/log/mongodb/` after 24h
- Disk usage: `df -h /var/log` < 50%
```

## Three Months Later

The fix has held up:

```bash
df -h /var/log
/dev/mapper/rootvg-var_loglv   6.0G  2.1G  3.9G  35% /var/log

ls -lh /var/log/mongodb/
-rw-r----- 1 mongodb mongodb  89M Dec 31 08:30 mongod.log
-rw-r----- 1 mongodb mongodb  91M Dec 30 23:59 mongod.log.1
-rw-r----- 1 mongodb mongodb  87M Dec 29 23:59 mongod.log.2.gz
-rw-r----- 1 mongodb mongodb  94M Dec 28 23:59 mongod.log.3.gz
# ... 7 days of logs, perfectly rotated
```

Disk usage: 35%. Logs rotating cleanly. No alerts. No incidents.

## Key Takeaways

**Before production:**
- ✓ Plan partition sizes based on actual usage
- ✓ Set up log rotation for all services
- ✓ Configure monitoring with alerts
- ✓ Have LVM for flexibility

**During incidents:**
- ✓ `truncate -s 0` frees space instantly
- ✓ `df` vs `du` helps diagnose
- ✓ `lsof | grep deleted` finds held files
- ✓ Check process status before `rm`

**After resolution:**
- ✓ Implement permanent fix, not just quick fix
- ✓ Document the incident
- ✓ Create runbooks
- ✓ Add monitoring to prevent recurrence

That 3 AM wake-up call taught me more about Linux filesystems, LVM, and production incident response than any tutorial could. MongoDB crashes are stressful, but they're also the best learning opportunities.

Now our monitoring catches disk space issues at 80% - long before they become 3 AM emergencies.

---

**Tools used:**
- `df -h` - Check filesystem usage
- `du -sh` - Check directory/file sizes
- `truncate -s 0` - Empty file without deleting
- `lvextend` - Extend logical volume
- `xfs_growfs` - Grow XFS filesystem online
- `logrotate` - Automated log rotation

**Resources:**
- [MongoDB Production Notes](https://docs.mongodb.com/manual/administration/production-notes/)
- [LVM Administrator's Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/logical_volume_manager_administration/index)
- [XFS Filesystem Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/ch-xfs)
