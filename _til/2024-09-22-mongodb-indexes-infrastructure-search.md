---
title: "Why our UI search was taking forever"
date: 2024-09-22
tags: [mongodb, search, performance, infrastructure]
---

Users kept complaining about search performance. With customers managing 1000+ VMs, they were constantly searching:

- VM by UUID: "550e8400-e29b-41d4-a716-446655440000"  
- VM by name: "prod-database-01"
- VM by IP: "10.24.3.156"
- Volume by UUID: "7b3f6d2a-9c8e-4f5a-b1d2-3e4f5a6b7c8d"
- Power host by hostname: "power-host-dal-03"
- Hypervisor hosts, network IDs... 

Every. Single. Search. Was. A. Collection. Scan.

```javascript
// What we were doing
db.vms.find({ $or: [
  { uuid: searchTerm },
  { name: searchTerm },
  { ip: searchTerm },
  { hostname: searchTerm }
]})
// Scanning 50k+ documents every time someone typed in the search box
```

The fix was actually simple:

```javascript
// Individual indexes for each searchable field
db.vms.createIndex({ "uuid": 1 })
db.vms.createIndex({ "name": 1 })  
db.vms.createIndex({ "ip": 1 })
db.vms.createIndex({ "hostname": 1 })

// Same for other collections
db.volumes.createIndex({ "uuid": 1 })
db.hosts.createIndex({ "hostname": 1 })
db.hypervisors.createIndex({ "hostname": 1 })
```

Search went from 15 seconds to under 100ms. Users are happy and less complaints.