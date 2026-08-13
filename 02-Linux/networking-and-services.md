# Linux Networking, Services & SSH

## Networking Commands

```bash
ip a                     # show network interfaces and IP addresses (modern)
ifconfig                 # legacy equivalent
ping google.com           # test connectivity
curl -I https://example.com  # test HTTP, show headers only
traceroute google.com     # trace the network path to a host
netstat -tulpn            # show listening ports and associated processes (legacy)
ss -tulpn                 # modern replacement for netstat
nslookup example.com      # DNS lookup
dig example.com            # detailed DNS lookup
```

## SSH (Secure Shell)

SSH is the standard way to remotely and securely administer Linux servers — directly relevant to managing EC2 instances.

```bash
ssh user@hostname                     # connect
ssh -i mykey.pem ec2-user@1.2.3.4      # connect using a specific private key
ssh-keygen -t ed25519 -C "my-email"    # generate a new SSH key pair
ssh-copy-id user@hostname               # copy your public key to a remote server
scp file.txt user@host:/remote/path     # copy a file over SSH
```

**Key-based vs. password authentication:** Key-based auth is strongly preferred in production — it's far harder to brute-force than a password, and it's the default method AWS uses for EC2 access.

## SSH Config File

`~/.ssh/config` lets you define shortcuts instead of typing full connection strings every time:

```
Host myserver
    HostName 1.2.3.4
    User ec2-user
    IdentityFile ~/.ssh/my-key.pem
    Port 22
```

Then simply: `ssh myserver`

## Managing Services (systemd)

Most modern Linux distributions (including Amazon Linux, Ubuntu) use `systemd` to manage services.

```bash
sudo systemctl status nginx     # check service status
sudo systemctl start nginx      # start a service
sudo systemctl stop nginx       # stop a service
sudo systemctl restart nginx    # restart a service
sudo systemctl enable nginx     # start automatically on boot
sudo systemctl disable nginx    # don't start on boot
journalctl -u nginx -f          # follow live logs for a service
```

## Package Management

| Distro Family | Package Manager | Example |
|---|---|---|
| Debian/Ubuntu | `apt` | `sudo apt install nginx` |
| RHEL/Amazon Linux/Fedora | `yum` / `dnf` | `sudo yum install nginx` |
| Alpine | `apk` | `apk add nginx` |

```bash
sudo apt update && sudo apt upgrade -y   # refresh package lists and upgrade installed packages
sudo apt install <package>                 # install a package
sudo apt remove <package>                  # remove a package
apt list --installed                        # list installed packages
```

## Firewalls (Linux-native, separate from AWS Security Groups)

```bash
sudo ufw status              # (Ubuntu) check firewall status
sudo ufw allow 22/tcp         # allow SSH
sudo ufw allow 80,443/tcp     # allow HTTP/HTTPS
sudo ufw enable                # enable the firewall
```

On an EC2 instance, both the **Security Group** (AWS-level) and any OS-level firewall (like `ufw`) can block traffic — both need to allow a port for it to be reachable.

## Best Practices

- Disable password authentication for SSH in production (`PasswordAuthentication no` in `/etc/ssh/sshd_config`), rely on keys only
- Keep systems patched — `sudo apt update && sudo apt upgrade` regularly, or automate with tools like `unattended-upgrades`
- Use `systemctl enable` for any service that should survive a reboot
- Monitor `journalctl` or `/var/log` for service failures rather than assuming things are running
- Never disable the firewall entirely — restrict it to exactly the ports needed

## Interview Prep

**Q: Why is key-based SSH authentication preferred over passwords?**
Keys are cryptographically much harder to brute-force than passwords, and they eliminate the risk of weak or reused passwords. AWS EC2 instances are provisioned with key-based auth by default for exactly this reason.

**Q: How would you troubleshoot a service that isn't responding on its expected port?**
Check the service is actually running (`systemctl status`), check what's listening on that port (`ss -tulpn`), check the OS firewall (`ufw status`), and if it's a cloud instance, check the Security Group rules too — the request has to clear every layer.

**Q: What's the difference between `systemctl start` and `systemctl enable`?**
`start` runs the service immediately but doesn't affect boot behavior. `enable` configures the service to start automatically on the next boot, but doesn't start it right now. In practice you often run both together for a service that should be running now and persist across reboots.

**Q: What's the difference between `apt update` and `apt upgrade`?**
`apt update` refreshes the local package index (metadata about what's available), it doesn't install anything. `apt upgrade` actually installs newer versions of packages already on the system, based on that refreshed index. Running `update` before `upgrade` is standard practice.
