+++
title = "Repairing Inconsistent Ceph PGs"
+++
My hyperconverged Ceph cluster was reporting an inconsistent placement group on one of my OSDs.  Proxmox doesn't provide any method in the UI to repair these, so I had to SSH into the node and run a repair using the Ceph CLI.

1. Check for inconsistent placement groups (PGs):
```bash
ceph pg dump pgs | grep "inconsistent"
```

2. The first column of the output lists the PGID (e.g. `13.d`).  The first part (`13`) is the ID of the pool.  You can see all the pools by ID and name with:
```bash
ceph osd pool ls detail | awk '{print $2, $3}'
```

3. You may get more info on the exact objects affected with these commands, where `<pool_name>` and `<pgid>` are from the outputs above.
```bash
rados list-inconsistent-pg <pool_name>
rados list-inconsistent-obj <pgid>
rados list-inconsistent-snapset <pgid>
```

4. Start a repair on the errored PGs with:
```bash
ceph pg repair <pgid>
```