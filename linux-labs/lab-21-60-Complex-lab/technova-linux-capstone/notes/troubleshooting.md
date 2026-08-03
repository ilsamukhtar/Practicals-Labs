# Troubleshooting Notes

Log any issues you hit while working through `CAPSTONE_LAB.md`, and how you fixed them.
Keep one entry per issue so this becomes a useful reference later.

---

## Template

### [Phase X] — short title of the issue

**Command that failed:**
```bash

```

**Error / symptom:**
```

```

**Root cause:**


**Fix:**
```bash

```

**Notes for next time:**


---

<!-- Example entry — delete once you add your own -->
## [Phase 6] — `mount` fails with "wrong fs type"

**Command that failed:**
```bash
sudo mount /dev/sdb1 /mnt/technova_data
```

**Error / symptom:**
```
mount: wrong fs type, bad option, bad superblock ...
```

**Root cause:**
Partition was never formatted — skipped `mkfs.ext4` after `fdisk`.

**Fix:**
```bash
sudo mkfs.ext4 /dev/sdb1
sudo mount /dev/sdb1 /mnt/technova_data
```

**Notes for next time:**
Always format a new partition before mounting it.
