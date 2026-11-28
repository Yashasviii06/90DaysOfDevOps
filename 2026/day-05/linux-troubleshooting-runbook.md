# Day 05 – Linux Troubleshooting Runbook: CPU, Memory, Disk, Network & Logs

> **Goal:** A short, repeatable checklist to follow when a service misbehaves. Capture evidence first, then act.

| | |
|---|---|
| **Target service** | `nginx` (web server on port 80) |
| **System** | Ubuntu 24.04 LTS on WSL2 (Windows 11) |
| **Why nginx** | It has a process, a port, an HTTP endpoint, and its own log files, so every check in this drill applies |
| **Setup** | `sudo apt install -y nginx sysstat` (`sysstat` provides `iostat`) |

<!-- Run every command on your own machine and replace the sample output with what you actually see. -->

---

## 📑 Contents
1. [The Runbook (one-page checklist)](#1-the-runbook-one-page-checklist)
2. [Environment Basics](#2-environment-basics)
3. [Filesystem Sanity](#3-filesystem-sanity)
4. [Snapshot: CPU & Memory](#4-snapshot-cpu--memory)
5. [Snapshot: Disk & IO](#5-snapshot-disk--io)
6. [Snapshot: Network](#6-snapshot-network)
7. [Logs Reviewed](#7-logs-reviewed)
8. [Quick Findings](#8-quick-findings)
9. [If This Worsens](#9-if-this-worsens)
10. [Reading the Signals (cheat sheet)](#10-reading-the-signals-cheat-sheet)
11. [Interview Questions](#11-interview-questions)

---

## 1. The Runbook (one-page checklist)

```
CAPTURE → INTERPRET → ACT → VERIFY
(evidence first, never restart blind)
```

| # | Check | Command | Healthy if… |
|---|---|---|---|
| 1 | Where am I? | `uname -a` · `cat /etc/os-release` | Expected host, kernel, OS |
| 2 | Can I write to disk? | `mkdir` + `cp` + `ls -l` in `/tmp` | File created without errors |
| 3 | Is the process alive? | `pgrep -a nginx` | master + worker processes listed |
| 4 | CPU / memory per process | `ps -o pid,ppid,pcpu,pmem,rss,etime,comm -C nginx` | Low %CPU, stable RSS |
| 5 | System CPU / load | `top -b -n 1 \| head -15` | Load < CPU cores (`nproc`) |
| 6 | System memory | `free -h` | Healthy `available`, swap mostly unused |
| 7 | Disk space | `df -h /` · `sudo du -sh /var/log` | Usage < 80%, logs not exploding |
| 8 | Disk IO / pressure | `vmstat 1 5` · `iostat -xz 1 2` | Low `wa`, low `%util` |
| 9 | Is it listening? | `sudo ss -tulpn \| grep nginx` | LISTEN on `:80` |
| 10 | Does it respond? | `curl -I http://localhost` | `HTTP/1.1 200 OK` |
| 11 | What do the logs say? | `journalctl -u nginx -n 50` · `tail -n 50 /var/log/nginx/error.log` | No new errors |

---

## 2. Environment Basics

```bash
$ uname -a
Linux LAPTOP-DEVOPS 6.6.87.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun  5 18:30:46 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```
📝 The kernel is the WSL2 Microsoft kernel on 64-bit (`x86_64`). Knowing the kernel matters when a bug or fix is kernel-specific.

```bash
$ cat /etc/os-release | head -4
PRETTY_NAME="Ubuntu 24.04.3 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.3 LTS (Noble Numbat)"
```
📝 Ubuntu 24.04 LTS. Package names and log paths depend on the distro, so I always confirm this first.

---

## 3. Filesystem Sanity

```bash
$ mkdir -p /tmp/runbook-demo
$ cp /etc/hosts /tmp/runbook-demo/hosts-copy && ls -l /tmp/runbook-demo
total 4
-rw-r--r-- 1 devops devops 412 Nov 27 11:05 hosts-copy
```
📝 Write and copy both worked, so the filesystem is not read-only or full. `&&` runs `ls` only if `cp` succeeds.

---

## 4. Snapshot: CPU & Memory

```bash
$ pgrep -a nginx
1830 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
1832 nginx: worker process
1833 nginx: worker process
```
📝 One master (PID 1830, runs as root, reads config) and worker processes (handle requests). Workers = CPU cores by default.

```bash
$ ps -o pid,ppid,pcpu,pmem,rss,etime,comm -C nginx
    PID    PPID %CPU %MEM   RSS     ELAPSED COMMAND
   1830       1  0.0  0.0  1764       16:41 nginx
   1832    1830  0.0  0.0  5888       16:41 nginx
   1833    1830  0.0  0.0  5888       16:41 nginx
```
📝 `-C nginx` selects processes by name. RSS is ~6 MB per worker and CPU is ~0%. `PPID 1830` confirms the workers are children of the master.

```bash
$ top -b -n 1 | head -n 12
top - 11:10:03 up 18 min,  1 user,  load average: 0.12, 0.06, 0.02
Tasks:  41 total,   1 running,  40 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.4 us,  0.3 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7838.2 total,   6862.1 free,    612.4 used,    589.0 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   7225.8 avail Mem
```
📝 Load 0.12 on 8 cores means the system is idle. `wa` is 0.0, so nothing is waiting on disk. Even during a burst of 200 `curl` requests, nginx workers stayed under 3% CPU.

```bash
$ free -h
               total        used        free      shared  buff/cache   available
Mem:           7.7Gi       612Mi       6.7Gi       3.1Mi       589Mi       7.1Gi
Swap:          2.0Gi          0B       2.0Gi
```
📝 Look at **`available`** (7.1 GiB), not `free`. Linux uses spare RAM as cache and gives it back when apps need it. Swap use is 0, so there's no memory pressure.

---

## 5. Snapshot: Disk & IO

```bash
$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdc       1007G  6.1G  950G   1% /
```
📝 Root filesystem is at 1%. **Alert threshold: > 80%.** A full disk makes services fail to write logs, PID files, or uploads.

```bash
$ sudo du -sh /var/log
38M	/var/log
$ sudo du -sh /var/log/* | sort -h | tail -n 3
1.2M	/var/log/apt
4.0K	/var/log/nginx
37M	/var/log/journal
```
📝 Logs total 38 MB, mostly the systemd journal. Nothing is growing out of control. `sort -h` sorts human-readable sizes (K < M < G).

```bash
$ vmstat 1 5
procs -----------memory---------- ---swap-- -----io---- -system-- -------cpu-------
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st gu
 0  0      0 7026720  45120 558332    0    0    52    31  128  210  1  0 99  0  0  0
 0  0      0 7026468  45120 558336    0    0     0     0  101  176  0  0 100 0  0  0
 0  0      0 7026468  45120 558336    0    0     0    12  108  189  0  0 100 0  0  0
```
📝 `r` (run queue) = 0, `b` (blocked) = 0, `si/so` (swap in/out) = 0, `wa` = 0. No CPU, memory, or IO pressure.

```bash
$ iostat -xz 1 2      # second report shown, columns trimmed
Device     r/s    w/s   rkB/s   wkB/s  r_await  w_await  aqu-sz  %util
sdc       0.00   2.00    0.00   12.00     0.00     0.50    0.00   0.20
```
📝 `%util` 0.2% and `w_await` 0.5 ms, so the disk is barely used. The first `iostat` report is the average since boot, so I always read the **second** one.

---

## 6. Snapshot: Network

```bash
$ sudo ss -tulpn | grep -E 'State|nginx'
Netid State  Recv-Q Send-Q Local Address:Port  Peer Address:Port Process
tcp   LISTEN 0      511          0.0.0.0:80         0.0.0.0:*     users:(("nginx",pid=1832,fd=5),("nginx",pid=1830,fd=5))
tcp   LISTEN 0      511             [::]:80            [::]:*     users:(("nginx",pid=1832,fd=6),("nginx",pid=1830,fd=6))
```
📝 nginx is listening on port 80 for IPv4 (`0.0.0.0`) and IPv6 (`[::]`), which means all interfaces. If it showed `127.0.0.1:80`, it would only be reachable from this machine.

```bash
$ curl -I http://localhost
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Fri, 27 Nov 2026 05:41:12 GMT
Content-Type: text/html
Content-Length: 615
Connection: keep-alive

$ curl -o /dev/null -s -w "HTTP %{http_code} in %{time_total}s\n" http://localhost
HTTP 200 in 0.001834s
```
📝 `-I` fetches headers only. I got 200 OK in under 2 ms. The second command is a one-line health check I can reuse in scripts.

```bash
$ ping -c 4 8.8.8.8
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 18.214/19.006/20.331/0.781 ms
```
📝 Outbound internet works: 0% loss, ~19 ms latency. This rules out a network problem on the host side.

---

## 7. Logs Reviewed

```bash
$ journalctl -u nginx -n 50 --no-pager
Nov 27 10:52:14 LAPTOP-DEVOPS systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
Nov 27 10:52:14 LAPTOP-DEVOPS systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.
```
📝 The journal only shows start/stop events from systemd. nginx writes request and error details to its **own** log files.

```bash
$ tail -n 3 /var/log/nginx/access.log
127.0.0.1 - - [27/Nov/2026:11:08:41 +0530] "HEAD / HTTP/1.1" 200 0 "-" "curl/8.5.0"
127.0.0.1 - - [27/Nov/2026:11:08:41 +0530] "GET / HTTP/1.1" 200 615 "-" "curl/8.5.0"
127.0.0.1 - - [27/Nov/2026:11:09:02 +0530] "GET /missing HTTP/1.1" 404 162 "-" "curl/8.5.0"
```
📝 Each line shows client IP, time, request, **status code**, size, and user agent. The 404 is from a test request I made on purpose.

```bash
$ sudo tail -n 50 /var/log/nginx/error.log
2026/11/27 11:09:02 [error] 1833#1833: *204 open() "/var/www/html/missing" failed (2: No such file or directory), client: 127.0.0.1, server: _, request: "GET /missing HTTP/1.1", host: "localhost"
```
📝 The only error is my intentional 404 test. There are no real errors, crashes, or permission problems.

---

## 8. Quick Findings

| Area | Status | Evidence |
|---|---|---|
| Process | ✅ Healthy | master + workers running, ~0% CPU, ~6 MB each |
| CPU | ✅ Idle | load 0.12 on 8 cores, 99% idle |
| Memory | ✅ Plenty | 7.1 GiB available, swap unused |
| Disk | ✅ OK | `/` at 1%, `/var/log` 38 MB |
| IO | ✅ OK | `wa` 0, `%util` < 1% |
| Network | ✅ OK | LISTEN on :80, `200 OK` in < 2 ms |
| Logs | ✅ Clean | only the intentional 404 test |

**Verdict:** nginx is healthy. No action needed.

---

## 9. If This Worsens

| Situation | Next step |
|---|---|
| **1. Service down or returning 5xx** | Collect evidence first (`systemctl status`, last 100 lines of `error.log`). Validate config with `sudo nginx -t`. If config changed, use `sudo systemctl reload nginx` (no downtime). Use `restart` only if reload fails. Then verify with `curl -I`. |
| **2. Errors unclear** | Increase log verbosity: set `error_log /var/log/nginx/error.log debug;` temporarily, reload, and watch live with `sudo tail -f /var/log/nginx/error.log`. Revert afterwards, because debug logs grow fast. |
| **3. Process hung / high CPU / slow** | Find the hot worker (`top`, then `P`), then trace it: `sudo strace -p <PID> -f -tt -o /tmp/nginx.strace` to see which system call it's stuck on. Check open files with `sudo lsof -p <PID>` and check for OOM kills with `journalctl -k \| grep -i oom`. |
| **Disk full (bonus)** | `sudo du -sh /var/log/* \| sort -h`, then rotate or compress logs (`logrotate`). Check `df -i` for inode exhaustion. |
| **Escalate when** | Still failing after the above, or users are impacted for more than 15 min. Share the evidence collected, not guesses. |

---

## 10. Reading the Signals (cheat sheet)

| Signal | Where | Worry when… |
|---|---|---|
| Load average | `top`, `uptime` | stays above `nproc` |
| `wa` (IO wait) | `top`, `vmstat` | consistently > 10–20% |
| `available` memory | `free -h` | close to 0 |
| Swap `si/so` | `vmstat` | non-zero and constant (memory pressure) |
| `r` run queue | `vmstat` | higher than CPU cores |
| `%util` / `await` | `iostat -x` | `%util` near 100% or `await` rising |
| Disk usage | `df -h` | > 80% |
| Inodes | `df -i` | > 80% (many small files) |
| HTTP status | `curl -I`, access log | 5xx = server problem, 4xx = client/request problem |

---

## 11. Interview Questions

<details>
<summary><b>Basic</b></summary>

**Q1. What is a runbook?**
A short, step-by-step checklist for handling a known type of incident: the commands to run, what "healthy" looks like, and what to do next. It reduces guesswork and speeds up recovery.

**Q2. `free` vs `available` in `free -h`?**
`free` is RAM that is completely unused. `available` is what applications can actually get, including cache Linux can release. Always judge memory by `available`.

**Q3. `df` vs `du`?**
`df` shows free/used space per **filesystem**. `du` shows how much space a **folder or file** uses. Use `df` to see that the disk is full, then `du` to find what filled it.

**Q4. What does `curl -I` do?**
Sends a request and shows only the response headers (status code, server, content type). It's a quick way to check whether a web service responds.

</details>

<details>
<summary><b>Intermediate</b></summary>

**Q5. What does `ss -tulpn` show?**
Listening sockets: `t` TCP, `u` UDP, `l` listening, `p` owning process, `n` numeric ports (no name lookup). It answers "which process is using port 80?"

**Q6. `0.0.0.0:80` vs `127.0.0.1:80`?**
`0.0.0.0` accepts connections on all network interfaces. `127.0.0.1` accepts only local connections from the same machine. This is a common reason a service "works locally but not from outside."

**Q7. High load average but CPU is mostly idle. What's going on?**
Processes are likely waiting on IO (state `D`), which also counts toward load. Check `wa` in `top`/`vmstat` and `%util`/`await` in `iostat`.

**Q8. Why read the second `iostat`/`vmstat` sample?**
The first line is the average since boot, which hides current activity. The following samples show what is happening right now.

**Q9. 4xx vs 5xx status codes?**
4xx = the request was wrong (404 not found, 403 forbidden). 5xx = the server failed (500 internal error, 502 bad gateway, 503 unavailable). 5xx usually means you need to investigate.

</details>

<details>
<summary><b>Advanced / Scenario</b></summary>

**Q10. `df -h` shows 100% full but `du` can't find the big files. Why?**
A deleted file is still held open by a running process, so its space isn't freed. Find it with `sudo lsof +L1` and restart or reload that process (or truncate the file through `/proc/<pid>/fd/`).

**Q11. `df -h` shows free space, but you get "No space left on device". Why?**
The filesystem has run out of **inodes** (too many small files). Check with `df -i`, then find and clean the directory with millions of files.

**Q12. A process suddenly disappeared with no error in its own log. What do you check?**
The OOM killer. Check `journalctl -k | grep -i -E "oom|killed process"` or `dmesg -T | grep -i oom`. The kernel kills processes when memory runs out.

**Q13. When would you use `strace`?**
When a process is hung or misbehaving and logs don't explain it. `strace -p <PID>` shows the system calls it's making live, for example stuck on a network connect or failing to open a file (`ENOENT`, `EACCES`).

**Q14. Restart vs reload: which first during an incident?**
Capture evidence first (status, logs), validate config (`nginx -t`), then prefer `reload` because it applies config without dropping connections. `restart` causes brief downtime and clears in-memory state, so it's a fallback.

**Q15. Why "capture evidence before acting"?**
A restart wipes in-memory state, open connections, and sometimes temporary logs. Without evidence you can't find the root cause, and the incident will happen again.

</details>

---

## Key Takeaways
- 🧾 A runbook turns panic into a checklist: **Capture → Interpret → Act → Verify**
- 🧠 Judge memory by **`available`**, not `free`
- 💽 `df` finds *that* a disk is full, and `du` finds *what* filled it
- 🌐 Listening (`ss`) ≠ responding (`curl`), so check both
- 📒 Many services (like nginx) log to **their own files**, not only the journal
- 🛑 Never restart blind. Evidence first.

---
*Part of my #90DaysOfDevOps journey · TrainWithShubham*
