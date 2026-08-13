# Linux File System & Permissions

## The Filesystem Hierarchy

| Directory | Purpose |
|---|---|
| `/` | Root of the entire filesystem |
| `/home` | User home directories |
| `/etc` | System-wide configuration files |
| `/var` | Variable data — logs, caches, spool files |
| `/var/log` | System and application log files |
| `/tmp` | Temporary files, cleared on reboot |
| `/usr` | User-installed software and libraries |
| `/bin`, `/sbin` | Essential binaries (user and system commands) |
| `/opt` | Optional/third-party software packages |
| `/proc` | Virtual filesystem exposing kernel/process info |

## File Permissions

Every file has three permission sets — **owner**, **group**, and **others** — each with **read (r)**, **write (w)**, and **execute (x)**.

```
-rwxr-xr-- 1 muzammil devs 1024 Aug 12 10:00 deploy.sh
```

Reading left to right: file type, owner perms, group perms, other perms.

| Symbol | Meaning |
|---|---|
| `-` | Regular file (`d` = directory, `l` = symlink) |
| `rwx` | Owner: read, write, execute |
| `r-x` | Group: read, execute (no write) |
| `r--` | Others: read only |

## Numeric (Octal) Permissions

Each permission has a value: `r=4`, `w=2`, `x=1`. Add them per group.

| Octal | Permission | Common Use |
|---|---|---|
| `755` | rwxr-xr-x | Executable scripts, directories |
| `644` | rw-r--r-- | Regular files (config, text) |
| `600` | rw------- | Private files (SSH keys) |
| `700` | rwx------ | Private directories/scripts |

```bash
chmod 755 deploy.sh       # set exact permissions
chmod +x deploy.sh        # add execute permission
chmod u+w,g-w file.txt    # symbolic form: user +write, group -write

chown muzammil:devs file.txt   # change owner and group
chgrp devs file.txt            # change group only
```

## Users, Groups & sudo

```bash
whoami                    # current user
id                         # user + group IDs
sudo useradd -m jane       # create user with home directory
sudo usermod -aG sudo jane # add jane to the sudo group
sudo passwd jane           # set/change password
groups jane                # list jane's groups
```

`sudo` grants temporary elevated privileges for a single command, rather than logging in as root directly — the safer default for administrative tasks.

## Essential Commands

```bash
ls -la              # list files, including hidden, with details
pwd                  # print working directory
cd /path/to/dir      # change directory
cp file1 file2       # copy
mv file1 newname     # move/rename
rm file              # remove file
rm -rf directory     # remove directory recursively (use with caution)
mkdir -p a/b/c        # create nested directories
find / -name "*.log" # search for files
grep "error" file.log # search inside file content
df -h                 # disk space usage (human-readable)
du -sh /var/log       # size of a directory
top / htop            # live process/resource monitor
ps aux                # list running processes
kill -9 <PID>          # force-kill a process
```

## Best Practices

- Avoid `chmod 777` — it grants everyone full access and is a common security red flag
- Use `sudo` for individual privileged commands instead of logging in as root
- Restrict SSH key files to `600` (`chmod 600 ~/.ssh/id_rsa`) — SSH refuses to use overly permissive keys
- Regularly check `/var/log` for errors and unusual activity
- Use `find` and `grep` together for troubleshooting instead of manually browsing large directories

## Interview Prep

**Q: What does `chmod 755` mean?**
It sets owner permissions to read/write/execute (`7`), and group/others to read/execute (`5` each) — a common setting for scripts and directories where the owner needs full control but others only need to run/view, not modify.

**Q: What's the difference between a hard link and a symbolic link?**
A hard link is a second directory entry pointing to the same inode (same data on disk) — deleting the original doesn't remove the data as long as a hard link exists. A symbolic link (symlink) is a separate file that just points to a path; if the original is deleted, the symlink breaks.

**Q: How would you find out what's consuming disk space on a server?**
Start with `df -h` to see overall usage per filesystem, then `du -sh /*` (or a specific directory) to narrow down which directory is largest, drilling deeper level by level until you find the specific files or logs responsible.

**Q: Why is `chmod 777` considered bad practice?**
It grants read, write, and execute permissions to the owner, group, and everyone else — meaning any user on the system (or process running as any user) can modify or execute the file. It's almost always a sign that proper ownership/permissions weren't configured, and it's a common target in security audits.
