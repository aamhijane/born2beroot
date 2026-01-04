_This project has been created as part of the 42 curriculum by ayamhija._

# Born2BeRoot

## Description

Born2BeRoot is a system administration project that introduces the fundamentals of virtualization and server configuration. The goal is to set up a secure virtual machine running Debian, following strict security policies and implementing proper system administration practices.

This project demonstrates understanding of:
- Virtual machine configuration and management
- Linux system administration fundamentals
- Security hardening (firewall, SSH, password policies)
- User and permission management
- Service configuration and monitoring
- Logical Volume Manager (LVM) and disk partitioning

The VM is configured with minimal services (no graphical interface), encrypted partitions using LVM, a hardened SSH server on port 4242, strict password policies, sudo configuration with logging, and automated system monitoring.

---

## Table of Contents

- [Operating System Choice](#operating-system-choice)
- [System Architecture](#system-architecture)
- [Installation Instructions](#installation-instructions)
- [Configuration Details](#configuration-details)
- [Technical Comparisons](#technical-comparisons)
- [Monitoring Script](#monitoring-script)
- [Bonus Features](#bonus-features)
- [Verification Commands](#verification-commands)
- [Resources](#resources)

---

## Operating System Choice

### Why Debian?

I chose **Debian 13 (trixie)** over Rocky Linux for this project.

**Advantages:**
- Recommended for beginners in system administration
- Simpler security module (AppArmor vs SELinux)
- Extensive documentation and large community
- Massive package repository (59,000+ packages via APT)
- Easier to configure and troubleshoot
- Stable and widely used in production environments

**Disadvantages:**
- Less common in enterprise/corporate environments compared to RHEL-based systems
- Rocky/RHEL skills are more valued in enterprise careers
- SELinux (used in Rocky) provides more granular security controls

---

## System Architecture

### System Information

```
OS: Debian GNU/Linux 13 (trixie)
Kernel: Linux 6.12.57+deb13-amd64
Architecture: x86_64
Hostname: ayamhija42
```

### Partition Structure (Mandatory)

```
NAME            SIZE   TYPE   MOUNTPOINT
sda              30G   disk
├─sda1          487M   part   /boot
├─sda2            1K   part
└─sda5         29.5G   part
  └─sda5_crypt 29.5G   crypt
    ├─LVMGroup-root    10G    lvm    /
    ├─LVMGroup-swap   2.3G    lvm    [SWAP]
    └─LVMGroup-home     5G    lvm    /home
```

### Partition Structure (Bonus)

```
NAME                   SIZE   TYPE   MOUNTPOINT
sda                    30G   disk
├─sda1                487M   part   /boot
├─sda2                  1K   part
└─sda5               29.5G   part
  └─sda5_crypt       29.5G   crypt
    ├─LVMGroup-root    10G    lvm    /
    ├─LVMGroup-swap   2.3G    lvm    [SWAP]
    ├─LVMGroup-home     5G    lvm    /home
    ├─LVMGroup-var      3G    lvm    /var
    ├─LVMGroup-srv      3G    lvm    /srv
    ├─LVMGroup-tmp      3G    lvm    /tmp
    └─LVMGroup-var-log  4G    lvm    /var/log
```

**Key Design Choices:**
- **Encrypted partitions**: Using LUKS encryption on sda5 for data security
- **LVM**: Provides flexibility for future partition resizing
- **Separate partitions**: Isolates system components for security and stability
- **/boot unencrypted**: Required for bootloader to access kernel

---

## Installation Instructions

### Prerequisites

- VirtualBox 7.0+ or UTM (for Apple Silicon)
- Debian 13 ISO image
- Minimum 1/2GB RAM, 8GB disk space (30GB recommended for bonus)

### Installation Steps

1. **Create Virtual Machine**
   ```
   - Name: Born2BeRoot
   - Type: Linux
   - Version: Debian (64-bit)
   - RAM: 1024 MB (minimum)
   - Disk: 30 GB VDI (dynamically allocated)
   ```

2. **Install Debian**
   - Choose manual partitioning
   - Set up encrypted LVM as shown in partition structure
   - Install only base system (NO graphical interface)
   - Install SSH server during installation

3. **Initial Configuration**
   ```bash
   # Update system
   apt update && apt upgrade -y

   # Install required packages
   apt install sudo ufw libpam-pwquality -y
   ```

4. **Network Configuration**
   - VirtualBox: NAT with port forwarding (Host 8080 → Guest 80, Host 4242 → Guest 4242)
   - OR Bridged adapter for direct network access

---

## Configuration Details

### 1. Hostname

```bash
# Current hostname: ayamhija42
# Configured in /etc/hostname and /etc/hosts
```

### 2. Users and Groups

```bash
# Primary user: ayamhija
# Groups: sudo, user42

# Commands used:
sudo adduser ayamhija
sudo usermod -aG sudo,user42 ayamhija
```

### 3. Password Policy

**Expiration settings** (`/etc/login.defs`):
```
PASS_MAX_DAYS   30    # Password expires every 30 days
PASS_MIN_DAYS   2     # Minimum 2 days before password change
PASS_WARN_AGE   7     # Warning 7 days before expiration
```

**Complexity requirements** (`/etc/pam.d/common-password`):
```
password requisite pam_pwquality.so retry=3 minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 maxrepeat=3 usercheck=1 difok=7 enforce_for_root
```

- Minimum 10 characters
- At least 1 uppercase, 1 lowercase, 1 digit
- Maximum 3 consecutive identical characters
- Cannot contain username
- At least 7 characters different from old password (non-root)

### 4. Sudo Configuration

**File**: `/etc/sudoers.d/sudo_config`

```
Defaults passwd_tries=3
Defaults badpass_message="Wrong password! Please try again."
Defaults logfile="/var/log/sudo/sudo.log"
Defaults log_input, log_output
Defaults requiretty
Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```

**Features:**
- 3 password attempts maximum
- Custom error message
- All sudo commands logged (input/output)
- TTY mode enabled (prevents automated attacks)
- Restricted secure path

### 5. SSH Configuration

**File**: `/etc/ssh/sshd_config`

```
Port 4242
PermitRootLogin no
```

**Security measures:**
- Non-standard port (reduces automated attacks)
- Root login disabled
- Only key-based or password authentication for regular users

### 6. UFW Firewall

```bash
sudo ufw enable
sudo ufw allow 4242    # SSH
sudo ufw allow 80      # HTTP (bonus - WordPress)
```

**Default policy:**
- Deny all incoming traffic
- Allow all outgoing traffic
- Explicit allow rules for required ports

### 7. AppArmor

```bash
# Status: Loaded and enforcing
# Profiles: 45+ active profiles
# Verification: sudo aa-status
```

---

## Technical Comparisons

### Debian vs Rocky Linux

| Feature | Debian | Rocky Linux |
|---------|--------|-------------|
| **Base** | Independent distribution | RHEL clone (enterprise) |
| **Package Manager** | APT (59k+ packages) | DNF/YUM (fewer packages) |
| **Security Module** | AppArmor (simpler) | SELinux (complex, powerful) |
| **Release Cycle** | ~2 years (stable) | ~6 months (tracks RHEL) |
| **Target Users** | General purpose, developers | Enterprise, production servers |
| **Learning Curve** | Beginner-friendly | Steeper learning curve |
| **Enterprise Support** | Community-driven | Commercial support available |

**My choice:** Debian is recommended for beginners in the subject and provides easier configuration while maintaining strong security.

### AppArmor vs SELinux

| Feature | AppArmor | SELinux |
|---------|----------|---------|
| **Approach** | Path-based | Label-based |
| **Complexity** | Simple, easy to learn | Complex, steep learning curve |
| **Configuration** | Text files in `/etc/apparmor.d/` | Security contexts and policies |
| **Default Mode** | Enforce mode | Multiple modes (enforcing/permissive/disabled) |
| **Granularity** | File paths | Fine-grained security contexts |
| **Used In** | Debian, Ubuntu, SUSE | RHEL, CentOS, Fedora, Rocky |

**My choice:** AppArmor is sufficient for this project's security requirements and easier to configure for beginners.

### UFW vs firewalld

| Feature | UFW | firewalld |
|---------|-----|-----------|
| **Full Name** | Uncomplicated Firewall | Dynamic Firewall Manager |
| **Backend** | iptables/nftables | iptables/nftables |
| **Configuration** | Simple command-line | Zone-based configuration |
| **Complexity** | Beginner-friendly | More features, steeper curve |
| **Dynamic Rules** | Requires reload | Dynamic (no reload needed) |
| **Used In** | Debian, Ubuntu | RHEL, CentOS, Fedora, Rocky |

**My choice:** UFW provides simple firewall management perfect for this project's needs.

### VirtualBox vs UTM

| Feature | VirtualBox | UTM |
|---------|------------|-----|
| **Platform** | x86/x64 (Intel/AMD) | ARM64 (Apple Silicon) + x86 |
| **License** | GPL (open source) | Apache 2.0 (open source) |
| **Performance** | Native virtualization | Native on ARM, QEMU on x86 |
| **Guest Additions** | Available | Limited support |
| **Ease of Use** | Mature, feature-rich | Simpler, macOS-focused |

**My choice:** VirtualBox for x86 systems (most common), UTM for Apple Silicon Macs.

---

## Monitoring Script

**File**: `/usr/local/bin/monitoring.sh`

The script displays system information every 10 minutes using `cron` and `wall`.

### Script Code

```bash
#!/bin/bash

arch=$(uname -a)
cpu_physical=$(lscpu | grep "Socket(s):" | awk '{ print $2 }')
vcpu=$(nproc)
mem_used=$(free -h | grep 'Mem:' | awk '{ print $3 }')
mem_total=$(free -h | grep 'Mem:' | awk '{ print $2 }')
mem_usage=$(free | grep "Mem" | awk '{ printf "(%.2f%%)", $3/$2*100 }')
disk_usage=$(df -h --total | grep "total" | awk '{ printf "%s/%s (%s)\n", $3, $2, $5 }')
cpu_load=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{ printf "%.1f%%\n", 100 - $1 }')
last_boot_date=$(last reboot --time-format iso | awk 'NR==1 { print  }' | sed -E 's/.*([0-9]{4}-[0-9]{2}-[0-9]{2}).*/\1/')
last_boot_time=$(last reboot --time-format iso | awk 'NR==1 { print  }' | sed -E 's/.*([0-9]{2}:[0-9]{2}).*/\1/')
lvm_use=$(cat /etc/fstab | grep "mapper" | wc -l | awk '{ if($1 >= "7") { printf "yes\n" } else { printf "no\n" } }')
conn_tcp=$(netstat -ant | grep "ESTABLISHED" | wc -l)
user_log=$(who | awk '{ print $1 }' | sort -u | wc -l)
net_ip=$(ip addr show enp0s3 | sed -n "s/.*inet \([0-9]\{1,3\}.[0-9]\{1,3\}.[0-9]\{1,3\}.[0-9]\{1,3\}\).*/\1/p")
net_mac=$(ip link show enp0s3 | sed -n "s/.*link\/ether \([0-9a-f:]\{17\}\).*/\1/p")
sudo_cmd=$(journalctl _COMM=sudo -q | grep 'COMMAND' | wc -l)

wall "
	#Architecture: $arch
	#CPU Physical: $cpu_physical
	#vCPU: $vcpu
	#Memory Usage: $mem_used/$mem_total $mem_usage
	#Disk Usage: $disk_usage
	#CPU load: $cpu_load
	#Last Boot: $last_boot_date $last_boot_time
	#LVM use: $lvm_use
	#Connections TCP: $conn_tcp ESTABLISHED
	#User log: $user_log
	#Network: IP $net_ip ($net_mac)
	#Sudo : $sudo_cmd cmd"
```

### Script Explanation

**Architecture and Kernel** (`uname -a`):
- Displays full system information including kernel version and architecture

**Physical CPUs** (`lscpu | grep "Socket(s):"`):
- Counts physical CPU sockets on the motherboard

**Virtual CPUs** (`nproc`):
- Counts logical CPU cores (including hyperthreading)

**Memory Usage** (`free`):
- Shows used/total RAM and calculates percentage
- Uses both `-h` (human-readable) and raw values for percentage

**Disk Usage** (`df -h --total`):
- Shows total disk usage across all partitions
- Displays used/total space and percentage

**CPU Load** (`top -bn1`):
- Calculates current CPU utilization
- Uses `sed` and `awk` to extract and calculate from idle percentage

**Last Boot** (`last reboot --time-format iso`):
- Shows date and time of last system reboot
- Extracts specific date/time format using `sed`

**LVM Status** (`cat /etc/fstab | grep "mapper"`):
- Checks if LVM is in use by counting mapper entries
- Returns "yes" if 7+ LVM volumes exist (bonus), otherwise "no"

**TCP Connections** (`netstat -ant`):
- Counts active ESTABLISHED TCP connections

**Logged Users** (`who`):
- Counts unique users currently logged in
- Uses `sort -u` to avoid duplicates

**Network Info** (`ip addr` and `ip link`):
- Extracts IPv4 address from network interface enp0s3
- Extracts MAC address from the same interface

**Sudo Commands** (`journalctl _COMM=sudo`):
- Counts total number of commands executed with sudo
- Reads from systemd journal logs

**Wall Command**:
- Broadcasts the message to all logged-in users' terminals
- This is how the information appears on all TTYs

### Cron Configuration

```bash
# Edit root's crontab
sudo crontab -e -u root

# Add this line to run every 10 minutes
*/10 * * * * /usr/local/bin/monitoring.sh
```

**Cron syntax breakdown:**
- `*/10` = every 10 minutes
- `*` = every hour
- `*` = every day
- `*` = every month
- `*` = every day of week

### Stop Monitoring Without Modifying Script

**Option 1: Stop cron service** (stops ALL cron jobs)
```bash
sudo systemctl stop cron
```

**Option 2: Disable the specific job** (recommended)
```bash
sudo crontab -e -u root
# Comment out the monitoring.sh line with #
# */10 * * * * /usr/local/bin/monitoring.sh
```

**Option 3: Remove all cron jobs for root**
```bash
sudo crontab -r -u root
```

**Option 4: Temporarily disable without editing**
```bash
# Make script non-executable
sudo chmod -x /usr/local/bin/monitoring.sh
# Cron will try to run it but fail silently
```

---

## Bonus Features

### 1. WordPress Website

**Services installed:**
- **lighttpd**: Lightweight web server (Apache/nginx excluded per subject)
- **MariaDB**: MySQL-compatible database
- **PHP**: Server-side scripting language

**Installation:**
```bash
sudo apt install lighttpd mariadb-server php-cgi php-mysql -y
sudo lighty-enable-mod fastcgi
sudo lighty-enable-mod fastcgi-php
sudo systemctl reload lighttpd
```

**WordPress setup:**
```bash
cd /var/www/html
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xzvf latest.tar.gz
sudo chown -R www-data:www-data wordpress
sudo chmod -R 755 wordpress
```

**Database configuration:**
```bash
sudo mysql -u root -p
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

**Access**: `http://VM_IP/wordpress` or `http://localhost:8080/wordpress` (with port forwarding)

### 2. Additional Service: fail2ban

**Purpose**: Intrusion prevention system that monitors logs and bans IPs with malicious behavior.

**Why useful:**
- Protects SSH from brute-force attacks
- Automatically bans repeat offenders (default: 3 failed attempts)
- Reduces server load from attack attempts
- Essential for any internet-facing server
- Logs all ban/unban actions for security auditing

**Installation:**
```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

**Configuration**: `/etc/fail2ban/jail.local`
```ini
[DEFAULT]
bantime = 10m
findtime = 10m
maxretry = 3

[sshd]
enabled = true
port = 4242
logpath = /var/log/auth.log
maxretry = 3
```

**Verify fail2ban is working:**
```bash
sudo systemctl status fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

---

## Verification Commands

### System Information
```bash
hostname                        # Check hostname (ayamhija42)
cat /etc/os-release            # OS version (Debian 13)
uname -r                        # Kernel version
uname -a                        # Full system info
```

### Partitions and LVM
```bash
lsblk                          # Partition structure
sudo pvs                        # Physical volumes
sudo vgs                        # Volume groups
sudo lvs                        # Logical volumes
df -h                           # Disk usage
```

### Users and Groups
```bash
groups                          # Current user's groups
groups ayamhija                 # Specific user's groups
getent group sudo               # Sudo group members
getent group user42             # user42 group members
id ayamhija                     # User ID and group IDs
cat /etc/passwd | grep ayamhija # User entry in passwd file
```

### Password Policy
```bash
sudo chage -l ayamhija          # Password aging info
cat /etc/login.defs | grep PASS # Expiration settings
cat /etc/pam.d/common-password  # Complexity rules
```

### Sudo Configuration
```bash
sudo cat /etc/sudoers.d/sudo_config  # Sudo rules
ls -la /var/log/sudo/                # Sudo logs directory
sudo cat /var/log/sudo/sudo.log      # View sudo logs
```

### SSH
```bash
sudo systemctl status ssh
sudo ss -tunlp | grep :4242
cat /etc/ssh/sshd_config | grep -E "Port|PermitRootLogin"
```

### Firewall
```bash
sudo ufw status verbose
sudo ufw status numbered
```

### AppArmor
```bash
sudo aa-status                  # Detailed status
sudo systemctl status apparmor  # Service status
```

### Services (Bonus)
```bash
sudo systemctl status lighttpd
sudo systemctl status mariadb
sudo systemctl status php8.2-fpm    # or php7.4-fpm depending on version
sudo systemctl status fail2ban
```

### Monitoring Script
```bash
bash /usr/local/bin/monitoring.sh   # Test script manually
sudo crontab -l -u root             # View cron jobs
sudo systemctl status cron          # Check cron service
```

### Network
```bash
ip addr                         # All network interfaces
ip addr show enp0s3             # Specific interface
ip link show enp0s3             # MAC address
hostname -I                     # IP addresses
ss -tunlp                       # Listening ports
netstat -ant | grep ESTABLISHED # Active connections
```

---

## Resources

### Official Documentation

- [Debian Documentation](https://www.debian.org/doc/) - Official Debian manuals and guides
- [VirtualBox User Manual](https://www.virtualbox.org/manual/) - Complete VirtualBox documentation
- [LVM HOWTO](https://tldp.org/HOWTO/LVM-HOWTO/) - Comprehensive LVM guide
- [OpenSSH Manual](https://www.openssh.com/manual.html) - SSH server and client documentation
- [UFW Documentation](https://help.ubuntu.com/community/UFW) - Uncomplicated Firewall guide
- [AppArmor Documentation](https://gitlab.com/apparmor/apparmor/-/wikis/Documentation) - AppArmor wiki and tutorials
- [Cron Documentation](https://man7.org/linux/man-pages/man5/crontab.5.html) - Crontab manual page

### Tutorials and Articles

- [Linux System Administration Basics](https://www.linode.com/docs/guides/linux-system-administration-basics/) - Fundamental sysadmin concepts
- [Understanding Linux File Permissions](https://www.redhat.com/sysadmin/linux-file-permissions-explained) - Permissions and ownership
- [Bash Scripting Tutorial](https://www.shellscript.sh/) - Shell scripting fundamentals
- [Cron Job Guide](https://www.digitalocean.com/community/tutorials/how-to-use-cron-to-automate-tasks-ubuntu-1804) - Scheduling tasks with cron
- [PAM Configuration](https://www.redhat.com/sysadmin/pluggable-authentication-modules-pam) - Understanding PAM for password policies
- [Sudo Security Best Practices](https://www.sudo.ws/posts/2021/01/sudo-security-policy-overview/) - Configuring sudo securely

### Video Resources

- [Linux Crash Course - System Administration](https://www.youtube.com/watch?v=ROjZy1WbCIA) - Comprehensive Linux basics
- [Understanding LVM](https://www.youtube.com/watch?v=MeltFN-bXrQ) - Visual explanation of Logical Volume Manager
- [SSH Hardening Tutorial](https://www.youtube.com/watch?v=Atbl7D_yPug) - Securing SSH servers

### AI Usage

**AI was used for the following tasks:**

1. **Concept Clarification**: Understanding complex topics like LVM, AppArmor, MAC vs DAC security models, and LUKS encryption
2. **Command Syntax Verification**: Confirming correct syntax for system administration commands (awk, sed, grep patterns)
3. **Troubleshooting**: Debugging issues with WordPress URL redirects, UFW rules, and service configuration conflicts
4. **Script Optimization**: Reviewing monitoring.sh script logic, optimizing awk/sed expressions, and ensuring efficient data extraction
5. **Documentation Structure**: Organizing this README with proper sections, markdown formatting, and technical comparisons
6. **Explanation Generation**: Creating clear explanations of technical concepts for the Technical Comparisons section

**AI was NOT used for:**
- Writing the actual configuration files (sudoers, sshd_config, UFW rules, PAM settings)
- Setting up the VM installation and manual disk partitioning
- Implementing security policies and password requirements
- Writing the monitoring script from scratch (written independently, then reviewed for optimization)
- Making architectural decisions (partition sizes, service choices)

**Learning approach**: 
I used AI as a teaching assistant to explain "why" certain configurations are needed and "how" specific commands work, rather than as a copy-paste solution provider. All configurations were implemented manually with full understanding of each component. When AI suggested commands, I researched their purpose, tested them in the VM, and verified their output before including them in the final setup.

**Verification process**:
Every AI-suggested command or configuration was:
- Researched in official documentation
- Tested in a safe environment first
- Understood before implementation
- Documented with comments explaining its purpose

This approach ensured genuine learning while leveraging AI as an efficient research and explanation tool.

---

## Project Evaluation

### Mandatory Requirements Met ✅

- ✅ Virtual machine created with VirtualBox
- ✅ Debian 13 (latest stable) installed
- ✅ No graphical interface (command-line only)
- ✅ At least 2 encrypted partitions using LVM (sda5_crypt with multiple LVMs)
- ✅ AppArmor running at startup and enforcing profiles
- ✅ SSH service on port 4242 with root login disabled
- ✅ UFW firewall active with only port 4242 open (mandatory)
- ✅ Hostname: ayamhija42 (login42 format)
- ✅ Strong password policy implemented (expiration and complexity)
- ✅ sudo configured with strict rules (logging, TTY mode, attempts limit)
- ✅ User ayamhija in both sudo and user42 groups
- ✅ monitoring.sh script running every 10 minutes via cron

### Bonus Requirements Met ✅

- ✅ Advanced partition structure with 7+ logical volumes (/root, /swap, /home, /var, /srv, /tmp, /var/log)
- ✅ Functional WordPress website with lighttpd, MariaDB, and PHP
- ✅ Additional useful service: fail2ban for intrusion prevention
- ✅ Appropriate firewall rules for bonus services (port 80 for HTTP)
- ✅ All bonus services properly configured and running at startup

---

## Author

**ayamhija** - 42 Network Student  
- Project: Born2BeRoot  
- School: 1337 (42 Network)  
- Date: January 2026  
- Login: ayamhija

---

## License

This project is part of the 42 Network curriculum. All configurations and scripts are provided for educational purposes.
