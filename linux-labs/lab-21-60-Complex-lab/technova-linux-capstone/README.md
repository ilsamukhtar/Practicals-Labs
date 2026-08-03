# 🐧 TechNova Linux Capstone Lab

A single, story-driven capstone lab that merges **41 individual Linux labs (Lab 20 → Lab 60)** from the *Linux Deep Dive* series into one continuous, real-world server setup — instead of 41 disconnected exercises.

> Scenario: you're the new Linux Sysadmin at a startup called **TechNova**. Over one working session you provision, secure, and maintain a fresh Linux server — packages, users, automation, everyday CLI tools, monitoring, storage, and security — all in one flow.

---

## 📌 What's in this repo

| File / Folder | Description |
|---|---|
| [`CAPSTONE_LAB.md`](./CAPSTONE_LAB.md) | The full merged lab guide — 7 phases, all commands, checkpoints |
| `screenshots/` | Terminal output screenshots per phase (add your own as you go) |
| `configs/` | Saved config files produced during the lab (logrotate, fstab entries, etc.) |
| `notes/troubleshooting.md` | Issues hit + fixes, for future reference |

---

## 🗺️ Lab Coverage Map

| Phase | Topic | Original Labs Covered |
|---|---|---|
| 1 | Server Prep | 20, 21, 22, 23, 24 |
| 2 | People & Access | 25, 26, 27, 30, 31 |
| 3 | Automation | 28, 29, 56, 57 |
| 4 | Daily Toolkit | 32, 33, 34, 35, 36, 46, 47, 48, 50, 51, 58 |
| 5 | Health Monitoring | 37, 53, 54, 59 |
| 6 | Storage Management | 41, 42, 43, 44, 45 |
| 7 | Lockdown / Security | 38, 39, 40, 52, 55, 60 |

**41/41 labs covered.**

---

## 🧰 Requirements

- A disposable Linux VM or container (Ubuntu/Debian recommended; a CentOS/Rocky VM too if you also want to test `yum/dnf` and SELinux)
- `sudo` privileges
- (Optional) a spare virtual disk for the partitioning/LVM phase
- Basic terminal familiarity

> ⚠️ Phases 6 and 7 touch disk partitions and firewall rules — always run this on a throwaway VM, never on your daily-driver machine.

---

## 🧩 Phase-by-Phase Breakdown

### Phase 1 — Server Prep
Get the box ready and know what you're working with: install `zip/unzip`, update packages via `apt` (or `yum/dnf`), and check hardware (CPU, RAM) and disk usage.

### Phase 2 — People & Access
Onboard a fake "team": create users and a group, set password aging policies, set up SSH key auth, and move files with SCP/SFTP.

### Phase 3 — Automation
Make the server do routine work on its own: schedule a recurring `crontab` job, run a one-time `at` job, inspect cron logs, and configure log rotation.

### Phase 4 — Daily Toolkit
Build muscle memory for everyday tools: edit files in `vi/vim`, filter logs with `grep`/regex, transform text with `sed`/`awk`, find files with `find`, customize `.bashrc`, use `history`, chain commands, and set up aliases.

### Phase 5 — Health Monitoring
Know if the server is happy: read system logs, check `uptime`, check disk space with `df`/`du`, and watch live processes with `top`/`htop`.

### Phase 6 — Storage Management
Manage disks properly: partition a spare disk, format and mount it, set up a swapfile, optionally build an LVM volume, and make the mount persistent via `/etc/fstab`.

> ⚠️ Destructive phase — only ever point this at a spare/virtual disk, never your root disk.

### Phase 7 — Lockdown / Security
Secure the server like it's going to production: manage services with `systemd`, enable the `UFW` firewall, add manual `iptables` rules, fetch files with `wget`/`curl`, securely delete a file with `shred`, and check/toggle `SELinux` mode (RHEL-family only).

---

## 🚀 How to Use This Lab

1. Clone this repo onto your VM:
   ```bash
   git clone https://github.com/ilsamukhtar/Practicals-Labs/
   cd linux-labs/lab-21-60-Complex-lab/technova-linux-capstone
   ```
2. Open [`CAPSTONE_LAB.md`](./CAPSTONE_LAB.md) and work through it **phase by phase**.
3. Tick off each phase's checkpoint before moving to the next.
4. Save terminal screenshots into `screenshots/` and any generated configs into `configs/` as you go.
5. Commit your progress after each phase so your Git history documents the whole build.

---

## ✅ Final Deliverable Checklist

- [ ] Packages updated & installed via apt/yum
- [ ] Users + group + password aging configured
- [ ] SSH key auth working, file transferred via SCP/SFTP
- [ ] Cron job + one-time `at` job + logrotate config running
- [ ] Logs filtered with grep/sed/awk/find/regex
- [ ] Custom `.bashrc` aliases + exported shell variable
- [ ] Uptime, df/du, top/htop health check documented
- [ ] Partition mounted via fstab + swapfile active
- [ ] UFW enabled, sshd managed via systemctl, file fetched via wget/curl, file securely shredded

---

## 📚 Source

Built from and consolidating Labs 20–60 of the *Linux Deep Dive (1–60)* lab series.

## 📝 License

Personal learning project — use freely for your own practice.
