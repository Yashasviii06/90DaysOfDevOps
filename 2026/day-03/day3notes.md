# Linux Commands Cheat Sheet

> A compact troubleshooting toolkit: 26 commands in 4 categories, each with a one-line usage note.
> **Legend:** `<PID>` = process ID, `<svc>` = service name (for example `nginx`).

---

## 1. Process Management
*Question answered: "What is running, and is it healthy?"*

| Command | Usage |
|---------|-------|
| `ps aux` | Snapshot of all processes with user, CPU, memory and state (STAT) |
| `top` | Live CPU and memory view per process (`q` to quit) |
| `pgrep -a <name>` | Find the PID of a process by name |
| `kill -15 <PID>` | Ask a process to stop gracefully (SIGTERM) |
| `kill -9 <PID>` | Force stop (SIGKILL). Last resort, no cleanup |
| `systemctl status <svc>` | Service state plus its most recent log lines |
| `free -h` | Memory used and available (look at *available*) |

## 2. File System
*Question answered: "Where is it, how big is it, and who can access it?"*

| Command | Usage |
|---------|-------|
| `ls -lah` | List files with permissions, sizes and hidden entries |
| `cd <dir>` | Change directory (`cd -` returns to the previous one) |
| `mkdir -p a/b/c` | Create nested directories in one step |
| `find . -name "*.log"` | Locate files by name under the current directory |
| `du -sh *` | Size of each item in the current directory |
| `df -h` | Free and used space per disk (a full disk breaks many things) |
| `chmod 640 <file>` | Set permissions: owner read/write, group read, others none |

## 3. Logs and Text Search
*Question answered: "What did the system say?"*

| Command | Usage |
|---------|-------|
| `less <file>` | Scroll a large file (`/word` to search, `q` to quit) |
| `tail -n 50 <file>` | Show the last 50 lines, where the newest events are |
| `tail -f app.log` | **Follow a log file live** while reproducing an issue |
| `grep -i error <file>` | Show only the lines containing "error" (case-insensitive) |
| `wc -l <file>` | Count lines, e.g. `grep -i error app.log \| wc -l` |
| `journalctl -u <svc> -f` | **Follow a systemd service's logs live** |

## 4. Networking Troubleshooting
*Question answered: "Where is the connection breaking?"*

| Command | Usage |
|---------|-------|
| `ping -c 4 google.com` | Check reachability, delay and packet loss |
| `ip addr` | Show this machine's IP addresses and interfaces |
| `dig example.com` | DNS lookup: which IP does this name resolve to? |
| `curl -I https://example.com` | Fetch headers only: the HTTP status code at a glance |
| `sudo ss -tulnp` | List listening ports and the process that owns each |
| `nc -zv <host> 80` | Test whether a specific port is open on a host |

> `dig` ships in the `dnsutils` package and `nc` in `netcat-openbsd`: `sudo apt install dnsutils netcat-openbsd`.

---

## 5. Troubleshooting Playbooks

### Website is not opening
```text
ping -> dig -> curl -I -> ss -tulnp -> systemctl status -> journalctl -u <svc> -f
```
| Step | Command | If it fails, suspect |
|------|---------|----------------------|
| 1 | `ping -c 4 google.com` | Network connectivity |
| 2 | `dig <your-site>` | DNS resolution |
| 3 | `curl -I <your-site>` | HTTP layer (look at the status code) |
| 4 | `sudo ss -tulnp` | App not listening on the expected port |
| 5 | `systemctl status <svc>` and `journalctl -u <svc> -f` | Service down or erroring |

### Server is slow
```text
top -> free -h -> df -h -> tail -f app.log
```
Check CPU, then memory, then disk space, then the application log.

### Disk is full
```text
df -h -> du -sh /var/* -> find . -name "*.log"
```
Find the full filesystem, then the biggest directory, then the files inside it.

---

## 6. Safety Rules
- Prefer `kill -15` before `kill -9`.
- `rm -r` has no undo. Re-read the path before pressing Enter.
- Use `man <command>` or `<command> --help` when unsure.
- `Tab` autocompletes, `Ctrl+R` searches history, `Ctrl+C` stops a running command.