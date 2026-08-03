# 🐧 Capstone Lab: Deploying & Hardening "TechNova" Linux Server
### (Merges Lab 20 → Lab 60 of *Linux Deep Dive* into one continuous, real-world scenario)

---

## 📖 Scenario

You've just been hired as the **Linux System Administrator** for a startup called **TechNova**. Your first assignment is to take a fresh Linux server (VM/container), and over one long working session:

1. Prepare it (packages, hardware/disk checks)
2. Onboard the team (users, groups, passwords, SSH access)
3. Automate routine work (cron, at, log rotation)
4. Master the day-to-day toolkit (vi/vim, grep, sed, awk, find, shell config)
5. Keep an eye on health (logs, uptime, df/du, top/htop)
6. Manage storage properly (partitions, mounts, swap, LVM, fstab)
7. Lock the server down (systemd, firewall, SELinux, secure deletion)

Every task below maps to one (or more) of Labs 20–60 from your document — nothing is invented, it's just re-sequenced into a story instead of 41 disconnected exercises.

> ⚠️ **Run this on a disposable VM or container** (VirtualBox, Multipass, WSL, or a cloud test instance) — Phases 6 and 7 touch partitions, swap, and firewall rules, and you don't want that on your daily-driver machine.

---

## 🧰 Prerequisites

- A Linux VM (Ubuntu/Debian recommended for `apt`/`ufw`; a CentOS/Rocky VM if you also want to genuinely test the `yum/dnf` and SELinux sections)
- `sudo` access
- A second small file/disk (real or virtual, e.g. `/dev/sdb` in a VM) for the partitioning/LVM phase — optional but recommended
- Basic comfort with a terminal

---

## Phase 1 — Server Prep (Labs 20, 21, 22, 23, 24)

**Goal:** get the box ready and know what you're working with.

```bash
# Lab 20 – zip/unzip
sudo apt install zip unzip -y
mkdir ~/technova && cd ~/technova
echo "TechNova setup log" > setup.log
zip setup.zip setup.log
unzip -l setup.zip

# Lab 21 – APT (Debian/Ubuntu)
sudo apt update
sudo apt list --upgradable
sudo apt install tree -y

# Lab 22 – YUM/DNF (RHEL/CentOS/Rocky — skip if you're on Ubuntu only)
sudo dnf check-update
sudo dnf install tree -y

# Lab 23 – Hardware info
lscpu
free -h
df -h

# Lab 24 – Disk usage & file size
du -sh ~/technova
du -h --max-depth=1 /var | sort -rh | head
```

**Checkpoint:** you can report CPU count, total RAM, free disk space, and the size of your `technova` folder.

---

## Phase 2 — People & Access (Labs 25, 26, 27, 30, 31)

**Goal:** onboard a fake "team" securely.

```bash
# Lab 25 – Users
sudo useradd -m alice
sudo useradd -m bob
sudo passwd alice

# Lab 26 – Groups
sudo groupadd devteam
sudo usermod -aG devteam alice
sudo usermod -aG devteam bob
getent group devteam

# Lab 27 – Password policies
sudo chage -l alice
sudo chage -M 90 -W 7 alice     # max age 90 days, warn 7 days before

# Lab 30 – SSH basics
sudo systemctl status ssh
ssh-keygen -t ed25519 -C "alice@technova"
ssh-copy-id alice@<server-ip>     # from your own machine, if testing remotely

# Lab 31 – SCP / SFTP
scp setup.zip alice@<server-ip>:~/
sftp alice@<server-ip>
# inside sftp: put/get a file, then `bye`
```

**Checkpoint:** `alice` and `bob` exist, both are in `devteam`, password aging is set, and you can SSH/SCP a file to the box.

---

## Phase 3 — Automation (Labs 28, 29, 56, 57)

**Goal:** make the server do routine work for you.

```bash
# Lab 28 – crontab
crontab -e
# add: 0 * * * * echo "hourly check $(date)" >> ~/technova/cron.log

# Lab 29 – at (one-time jobs)
echo "echo 'one-time backup run' >> ~/technova/at.log" | at now + 2 minutes
atq

# Lab 56 – Cron log inspection
grep CRON /var/log/syslog | tail          # Debian/Ubuntu
# journalctl -u cron --since today        # alternative via systemd

# Lab 57 – Log rotation
sudo nano /etc/logrotate.d/technova
```
```
~/technova/cron.log {
    weekly
    rotate 4
    compress
    missingok
}
```

**Checkpoint:** cron entry is running hourly, `at` job fired once, and you have a working logrotate config for your own log file.

---

## Phase 4 — The Daily Toolkit (Labs 32, 33, 34, 35, 36, 46, 47, 48, 50, 51, 58)

**Goal:** build muscle memory for the tools you'll use every day.

```bash
# Lab 32 – vi/vim
vi ~/technova/notes.txt        # i to insert, Esc, :wq to save+quit

# Lab 50 – Shell variables
export TECHNOVA_ENV="staging"
echo $TECHNOVA_ENV

# Lab 33 & 51 – grep & regex
grep -i "error" /var/log/syslog | head
grep -E "^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" /var/log/auth.log

# Lab 34 – sed
sed 's/staging/production/' ~/technova/notes.txt

# Lab 35 – awk
df -h | awk '{print $1, $5}'

# Lab 36 – find
find ~/technova -name "*.log" -mtime -1

# Lab 46 – bash_profile vs bashrc
echo 'alias ll="ls -la"' >> ~/.bashrc
source ~/.bashrc

# Lab 47 – history
history | tail -20
history | grep ssh

# Lab 48 – command chaining
sudo apt update && sudo apt upgrade -y || echo "update failed"

# Lab 58 – Aliases
alias techlogs='cd ~/technova && ls -la'
```

**Checkpoint:** you can filter logs with grep/regex, transform text with sed/awk, find files by date, and you've got working aliases + a customized `.bashrc`.

---

## Phase 5 — Health Monitoring (Labs 37, 53, 54, 59)

**Goal:** know if the server is happy.

```bash
# Lab 37 – System logs
sudo tail -f /var/log/syslog          # Ctrl+C to stop
journalctl -p err -b

# Lab 53 – Uptime
uptime

# Lab 54 – df & du
df -h
du -sh /var/log

# Lab 59 – top / htop
top
sudo apt install htop -y
htop
```

**Checkpoint:** you can state current uptime, load average, disk free %, and identify the top CPU/memory consuming process.

---

## Phase 6 — Storage Management (Labs 41, 42, 43, 44, 45)

**Goal:** properly manage disks — the part that actually needs a spare/virtual disk.

```bash
# Lab 41 – Partitioning (use a SECOND disk, e.g. /dev/sdb, never your root disk!)
sudo fdisk -l
sudo fdisk /dev/sdb        # n -> new partition, w -> write

# Lab 42 – Mount/unmount
sudo mkfs.ext4 /dev/sdb1
sudo mkdir /mnt/technova_data
sudo mount /dev/sdb1 /mnt/technova_data
df -h | grep technova_data
sudo umount /mnt/technova_data

# Lab 43 – Swap
sudo fallocate -l 512M /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
swapon --show

# Lab 44 – LVM (needs another disk, e.g. /dev/sdc)
sudo pvcreate /dev/sdc
sudo vgcreate technova_vg /dev/sdc
sudo lvcreate -L 2G -n technova_lv technova_vg
sudo mkfs.ext4 /dev/technova_vg/technova_lv

# Lab 45 – /etc/fstab
sudo blkid /dev/sdb1
sudo nano /etc/fstab
# add: /dev/sdb1  /mnt/technova_data  ext4  defaults  0  2
sudo mount -a          # test fstab without rebooting
```

**Checkpoint:** you have a mounted partition that survives reboot (fstab), an active swapfile, and (optionally) an LVM logical volume.

> ⚠️ **This phase is destructive to whatever disk you point it at.** Triple-check device names with `lsblk` before running `fdisk`/`mkfs` on anything.

---

## Phase 7 — Lock It Down (Labs 38, 39, 40, 52, 55, 60)

**Goal:** secure the server like it's going to production.

```bash
# Lab 38 – systemd
systemctl list-units --type=service --state=running
sudo systemctl restart sshd
sudo systemctl status sshd

# Lab 39 – UFW firewall
sudo ufw status
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status verbose

# Lab 40 – iptables (deeper/manual firewall control)
sudo iptables -L -v -n
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Lab 52 – wget / curl
wget https://example.com/index.html -O ~/technova/index_test.html
curl -I https://example.com

# Lab 55 – Secure deletion
shred -u -z ~/technova/setup.log

# Lab 60 – SELinux (RHEL/CentOS/Rocky only)
getenforce
sudo setenforce 0        # permissive, for testing only
sudo setenforce 1         # back to enforcing
```

**Checkpoint:** UFW is active with SSH allowed, sshd was restarted cleanly via systemd, you fetched a file with wget/curl, securely shredded a file, and (on RHEL-family) you can read/toggle SELinux mode.

---

## ✅ Final Deliverable Checklist

- [ ] Packages updated & installed via apt/yum
- [ ] `alice`/`bob` users + `devteam` group + password aging
- [ ] SSH key auth working, file moved via SCP/SFTP
- [ ] Cron job + one-time `at` job + logrotate config running
- [ ] Logs filtered with grep/sed/awk/find/regex
- [ ] Custom `.bashrc` aliases + exported shell variable
- [ ] Uptime, df/du, top/htop health check documented
- [ ] Partition mounted via fstab + swapfile active
- [ ] UFW enabled, sshd managed via systemctl, one file wget'd, one file shredded

---

## 📦 Suggested GitHub Repo Structure

```
technova-linux-capstone/
├── README.md              <- this file
├── screenshots/           <- terminal screenshots per phase
├── configs/
│   ├── logrotate/technova
│   └── fstab.snippet
└── notes/
    └── troubleshooting.md
```

---
