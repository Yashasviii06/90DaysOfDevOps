# Day 04: Linux Practice, Processes and Services

> **Fill-in guide (delete this box before pushing):** every `<...>` below must be replaced with the **real output from my own machine**. Run each command, paste what I see, then write one line of what I understood in my own words.

## Environment

| Item | Value |
|------|-------|
| OS | `<output of: cat /etc/os-release \| head -3>` |
| Shell and user | `<output of: whoami>` |
| Service I inspected | `cron` (`<or ssh / docker>`) |

---

## 1. Process Checks

### 1.1 All running processes
```bash
ps aux | head -10
```
```text
<paste output here>
```
**What I observed:** `<e.g. which process has PID 1, and what is the STAT column showing?>`

### 1.2 Live resource usage (one snapshot)
```bash
top -b -n 1 | head -15
```
```text
<paste output here>
```
**What I observed:** `<e.g. load average, which process uses the most CPU or memory>`

### 1.3 Find my service's process
```bash
pgrep -a cron
```
```text
<paste output here>
```
**What I observed:** `<e.g. the PID of the service>`

---

## 2. Service Checks

### 2.1 Status of one service
```bash
systemctl status cron --no-pager
```
```text
<paste output here>
```
**What I observed:** `<e.g. Active line, Loaded line (enabled or disabled), Main PID>`

### 2.2 All running services
```bash
systemctl list-units --type=service --state=running --no-pager | head -15
```
```text
<paste output here>
```
**What I observed:** `<e.g. how many services are running and which ones I recognise>`

### 2.3 Will it start at boot?
```bash
systemctl is-enabled cron
```
```text
<paste output here>
```
**What I observed:** `<enabled or disabled, and what that means>`

---

## 3. Log Checks

### 3.1 Service logs from journald
```bash
journalctl -u cron -n 20 --no-pager
```
```text
<paste output here>
```
**What I observed:** `<e.g. what kind of lines cron writes, and the time of the last entry>`

### 3.2 Last lines of a log file
```bash
tail -n 20 /var/log/dpkg.log
```
```text
<paste output here>
```
**What I observed:** `<e.g. what this file records>`

> If `/var/log/syslog` exists on my system, I can use `tail -n 50 /var/log/syslog` here instead.

---

## 4. Mini Troubleshooting: "The service stopped. What now?"

I stopped the service on purpose, checked what the system showed, then fixed it. (Only on my own practice machine.)

| Step | Command | What I saw |
|------|---------|------------|
| 1. Stop it | `sudo systemctl stop cron` | `<paste or describe>` |
| 2. Check status | `systemctl status cron --no-pager` | `<e.g. Active: inactive (dead)>` |
| 3. Process still there? | `pgrep -a cron` | `<e.g. no output>` |
| 4. Read the logs | `journalctl -u cron -n 10 --no-pager` | `<e.g. the "Stopping" and "Stopped" lines>` |
| 5. Start it again | `sudo systemctl start cron` | `<paste or describe>` |
| 6. Verify | `systemctl status cron --no-pager` | `<e.g. Active: active (running)>` |

**My troubleshooting flow:**
```text
status -> process check -> logs -> fix (start/restart) -> verify
```

---

## 5. What I Learned
- `<one line in my own words about processes>`
- `<one line about what systemctl status tells me>`
- `<one line about how logs help when a service stops>`

## 6. Problems I Faced and How I Fixed Them
- `<e.g. systemctl said "not booted with systemd" on WSL. Fixed by adding systemd=true in /etc/wsl.conf and running wsl --shutdown>`
- `<or write "none">`