# Born2beRoot - System Administration & Virtualization

[![License: MIT](https://img.shields.io/badge/license-MIT-green?style=for-the-badge)](LICENSE)
[![Language Bash](https://img.shields.io/badge/language-Bash-blue?style=for-the-badge)](https://www.gnu.org/software/bash/)
[![42 Kocaeli](https://img.shields.io/badge/42-Kocaeli-black?style=for-the-badge)](https://www.42kocaeli.com.tr)

> A comprehensive system administration project involving virtual machine setup, LVM configuration, security hardening, and automated system monitoring. This project demonstrates deep knowledge of Linux system administration, security policies, and server management.

## 📖 Table of Contents
* [About The Project](#-about-the-project)
* [Key Features & Technical Skills](#-key-features--technical-skills)
* [System Configuration](#-system-configuration)
  * [LVM Partitioning](#lvm-partitioning)
  * [Security Hardening](#security-hardening)
  * [Monitoring Script](#monitoring-script)
* [Technical Implementation](#-technical-implementation)
* [How To Use](#-how-to-use)
* [What I Learned](#-what-i-learned)
* [Project Requirements](#-project-requirements)
* [License](#-license)
* [Contact](#-contact)

## 🎯 About The Project

Born2beRoot is a system administration project from **42 School's curriculum** that introduces **virtualization** and **Linux server management**. The project involves setting up a secure Debian server with strict configuration rules, including encrypted partitions, firewall setup, SSH hardening, and automated system monitoring.

**Why This Project Matters:**
- Provides hands-on experience with **Linux server configuration** and virtualization
- Teaches essential **security practices** including encrypted LVM, SSH hardening, and firewall configuration
- Develops skills in **bash scripting** for system automation and monitoring
- Introduces **storage management** concepts (LVM, LUKS encryption)
- Builds foundation for **DevOps** and **system administrator** roles with practical server setup

The project includes setting up VirtualBox with Debian, configuring encrypted LVM partitions, implementing strict password policies, hardening SSH on port 4242, configuring UFW firewall, and creating automated monitoring scripts.

## 🔧 Key Features & Technical Skills

| Category | Skills Demonstrated |
|----------|-------------------|
| **Virtualization** | VirtualBox setup, VM configuration, snapshot management |
| **Storage Management** | LVM (Logical Volume Manager), encrypted partitions, disk management |
| **Security** | SSH hardening, UFW firewall, AppArmor, password policies |
| **User Management** | sudo configuration, group management, user creation |
| **Bash Scripting** | System monitoring, automated reporting, cron jobs |
| **System Administration** | Service configuration, system policies, hostname management |
| **Networking** | Port configuration, TCP connections, IP/MAC address management |
| **Package Management** | apt/aptitude usage, service installation and configuration |
| **Web Services** | WordPress deployment, lighttpd, MariaDB, PHP configuration |

## ⚙️ System Configuration

### LVM Partitioning

The project requires setting up **encrypted LVM partitions** for enhanced security. Below is the mandatory partition structure:

**Mandatory Structure:**

Example partition structure:
```bash
wil@wil:~$ lsblk
NAME                MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                   8:0    0    8G  0 disk 
├─sda1                8:1    0  487M  0 part /boot
├─sda2                8:2    0    1K  0 part 
└─sda5                8:5    0  7.5G  0 part 
  └─sda5_crypt      254:0    0  7.5G  0 crypt 
    ├─wil--vg-root  254:1    0  2.8G  0 lvm  /
    ├─wil--vg-swap_1 254:2    0  976M  0 lvm  [SWAP]
    └─wil--vg-home  254:3    0  3.8G  0 lvm  /home
sr0                  11:0    1 1024M  0 rom  
wil@wil:~$
```

**Bonus Structure - Implemented** (More complex with additional logical volumes):
```bash
# lsblk
NAME                 MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                    8:0    0  30.8G  0 disk 
├─sda1                 8:1    0   500M  0 part /boot
├─sda2                 8:2    0     1K  0 part 
└─sda5                 8:5    0  30.3G  0 part 
  └─sda5_crypt       254:0    0  30.3G  0 crypt 
    ├─LVMGroup-root  254:1    0    10G  0 lvm  /
    ├─LVMGroup-swap  254:2    0   2.3G  0 lvm  [SWAP]
    ├─LVMGroup-home  254:3    0     5G  0 lvm  /home
    ├─LVMGroup-var   254:4    0     3G  0 lvm  /var
    ├─LVMGroup-srv   254:5    0     3G  0 lvm  /srv
    ├─LVMGroup-tmp   254:6    0     3G  0 lvm  /tmp
    └─LVMGroup-var--log 254:7 0    4G   0 lvm  /var/log
sr0                   11:0    1  1024M  0 rom
```

### Security Hardening

#### SSH Configuration
```bash
Port 4242                    # Custom port (not default 22)
PermitRootLogin no          # Root login disabled
PasswordAuthentication yes  # Password auth enabled
```

#### UFW Firewall
```bash
# Only port 4242 open
ufw allow 4242/tcp
ufw enable
ufw status
```

#### Password Policy
**Requirements implemented:**
- Password expires every **30 days**
- Minimum **2 days** before password change
- Warning **7 days** before expiration
- Minimum length: **10 characters**
- Must contain: uppercase, lowercase, and number
- Maximum **3 consecutive identical characters**
- Cannot contain username
- Must have **7 new characters** (non-root only)

**Configuration files:**
```bash
/etc/login.defs          # Password aging
/etc/pam.d/common-password  # Password complexity
```

#### Sudo Configuration
**Security rules implemented:**
- Maximum **3 authentication attempts**
- Custom error message for wrong password
- All sudo commands logged to `/var/log/sudo/`
- **TTY mode** enabled
- Restricted **secure_path**

**Configuration file:**
```bash
/etc/sudoers.d/sudo_config
```

### Monitoring Script

A bash script (`monitoring.sh`) that displays system information every 10 minutes using `cron` and `wall`.

**monitoring.sh Implementation:**
```bash
#!/bin/bash

# ARCH
arch=$(uname -a)

# CPU PHYSICAL
cpuf=$(grep "physical id" /proc/cpuinfo | wc -l)

# CPU VIRTUAL
cpuv=$(grep "processor" /proc/cpuinfo | wc -l)

# RAM
ram_total=$(free -m | awk 'NR==2{print $2}')
ram_use=$(free -m | awk 'NR==2{print $3}')
ram_percent=$(free -m | awk 'NR==2{printf("%.2f"), $3*100/$2}')

# DISK
disk_total=$(df -Bg | grep '^/dev/' | grep -v '/boot$' | awk '{ft += $2} END {print ft}')
disk_use=$(df -Bm | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} END {print ut}')
disk_percent=$(df -Bm | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} {ft+= $2} END {printf("%d"), ut/ft*100}')

# CPU LOAD
cpu_load=$(top -bn1 | grep '^%Cpu' | cut -c 9- | xargs | awk '{printf("%.1f%%"), $1 + $3}')

# LAST BOOT
last_boot=$(who -b | awk '$1 == "system" {print $3 " " $4}')

# LVM USE
lvm_use=$(if [ $(lsblk | grep "lvm" | wc -l) -eq 0 ]; then echo no; else echo yes; fi)

# TCP CONNEXIONS
tcp=$(ss -neopt state established | wc -l)

# USER LOG
ulog=$(users | wc -w)

# NETWORK
ip=$(hostname -I)
mac=$(ip link | grep "link/ether" | awk '{print $2}')

# SUDO
cmnd=$(journalctl _COMM=sudo | grep COMMAND | wc -l)

wall "	#Architecture: $arch
	#CPU physical : $cpuf
	#vCPU : $cpuv
	#Memory Usage: $ram_use/${ram_total}MB ($ram_percent%)
	#Disk Usage: $disk_use/${disk_total}GB ($disk_percent%)
	#CPU load: $cpu_load
	#Last boot: $last_boot
	#LVM use: $lvm_use
	#Connections TCP : $tcp ESTABLISHED
	#User log: $ulog
	#Network: IP $ip ($mac)
	#Sudo : $cmnd cmd"
```

**Cron Configuration:**
```bash
# Run monitoring.sh every 10 minutes
*/10 * * * * /path/to/monitoring.sh
```

## 🔍 Technical Implementation

### Core Components

**1. Virtualization Setup**
- **Platform:** VirtualBox (or UTM for ARM architecture)
- **OS:** Debian (latest stable version)
- **No GUI:** Minimal installation without X.org
- **Boot mode:** Configured for automatic startup

**2. Disk Encryption & LVM**
- **Encryption:** LUKS (Linux Unified Key Setup)
- **Volume Manager:** LVM for flexible disk management
- **Partitioning:** Logical volumes for system directories
- **Benefits:** Security through encryption, flexibility through LVM

**3. Network Security**
- **SSH:** Configured on port 4242, root login disabled
- **Firewall:** UFW (Uncomplicated Firewall) with strict rules
- **Services:** Minimal services running (security best practice)

**4. Access Control**
- **AppArmor:** Mandatory Access Control enabled at startup
- **Sudo:** Restricted with logging and attempt limits
- **Users:** Proper group management (user42, sudo groups)

**5. Automation**
- **Cron jobs:** Scheduled tasks for monitoring
- **Bash scripting:** System metric collection and reporting
- **Wall command:** Broadcasting to all terminals

### Notable Challenges

**Challenge 1: LVM with Encryption**
- Setting up encrypted partitions during Debian installation
- Configuring logical volumes for different mount points
- Ensuring proper boot sequence with encrypted root

**Challenge 2: Password Policy Implementation**
- Configuring PAM (Pluggable Authentication Modules)
- Testing password complexity rules
- Applying policies to existing and new users

**Challenge 3: Sudo Security**
- Understanding sudoers file syntax
- Setting up comprehensive logging
- Balancing security with usability

**Challenge 4: Monitoring Script**
- Parsing system files (`/proc`, `/sys`)
- Handling different command outputs
- Ensuring script reliability and efficiency

## 🚀 How To Use

### Prerequisites

- **VirtualBox** (or UTM for Mac M1/M2)
- **Debian** (latest stable) or **Rocky Linux** ISO
- Minimum 8GB disk space for VM
- Basic understanding of Linux commands

### Setup Instructions

**1. Create Virtual Machine:**
```bash
# VirtualBox
- Open VirtualBox
- New → Name: "Born2beRoot", Type: Linux, Version: Debian (64-bit)
- Memory: 1024 MB (1 GB minimum)
- Create virtual hard disk: VDI, Dynamically allocated, 8-30 GB
```

**2. Install Debian with LVM:**
- Boot from Debian ISO
- Choose "Install" (not graphical install)
- Follow partitioning guide for encrypted LVM
- Set up user accounts and passwords according to project requirements

**3. Configure SSH (Port 4242):**
```bash
# Edit SSH configuration
sudo nano /etc/ssh/sshd_config

# Change port and disable root login
Port 4242
PermitRootLogin no

# Restart SSH service (use 'sshd' if 'ssh' doesn't work)
sudo systemctl restart ssh
```

**4. Configure UFW Firewall:**
```bash
# Install and enable UFW
sudo apt install ufw
sudo ufw allow 4242/tcp
sudo ufw enable
sudo ufw status
```

**5. Set Up Password Policy:**
```bash
# Install password quality checking library
sudo apt install libpam-pwquality

# Configure password aging (in /etc/login.defs):
# PASS_MAX_DAYS 30
# PASS_MIN_DAYS 2
# PASS_WARN_AGE 7

# Configure password complexity (in /etc/pam.d/common-password):
# Add: minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 maxrepeat=3
# See Security Hardening section for detailed requirements
```

**6. Configure Sudo:**
```bash
# Create log directory first
sudo mkdir -p /var/log/sudo

# Create sudo configuration file
sudo visudo -f /etc/sudoers.d/sudo_config

# Add these security rules:
# Defaults passwd_tries=3
# Defaults badpass_message="Custom error message"
# Defaults logfile="/var/log/sudo/sudo.log"
# Defaults requiretty
# Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
# See Security Hardening section for complete configuration
```

**7. Deploy monitoring.sh:**
```bash
# Clone repository
git clone https://github.com/beratbosnak/Born2beRoot.git
cd Born2beRoot

# Make script executable
chmod +x monitoring.sh

# Test script
sudo ./monitoring.sh

# Set up cron job (every 10 minutes)
sudo crontab -e
# Add line (replace /home/username/Born2beRoot with your actual path):
*/10 * * * * /home/username/Born2beRoot/monitoring.sh
```

### Usage Example

**Running monitoring.sh manually:**
```bash
# Navigate to project directory
cd Born2beRoot

# Execute with sudo (required for system information)
sudo ./monitoring.sh
```

**Example output:**
```
Broadcast message from root@wil (tty1) (Sun Apr 25 15:45:00 2021):

#Architecture: Linux wil 4.19.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64 GNU/Linux
#CPU physical : 1
#vCPU : 1
#Memory Usage: 74/987MB (7.50%)
#Disk Usage: 1009/2Gb (49%)
#CPU load: 6.7%
#Last boot: 2021-04-25 14:45
#LVM use: yes
#Connections TCP : 1 ESTABLISHED
#User log: 1
#Network: IP 10.0.2.15 (08:00:27:51:9b:a5)
#Sudo : 42 cmd
```

**Accessing VM via SSH:**
```bash
# First, find VM's IP address (run this inside VM)
ip addr show

# From host machine (replace with actual VM IP)
ssh username@VM_IP_ADDRESS -p 4242

# Example:
ssh parallels@10.211.55.3 -p 4242

# Note: If using NAT networking in VirtualBox, you need to set up port forwarding:
# VirtualBox → Settings → Network → Advanced → Port Forwarding
# Host Port: 4242 → Guest Port: 4242
# Then use: ssh username@localhost -p 4242
```

**Stopping cron job (for defense):**
```bash
# Temporarily stop cron service
sudo systemctl stop cron

# Or remove cron job
sudo crontab -e
# Comment out or delete the monitoring.sh line
```

## 💡 What I Learned

This project provided comprehensive experience in Linux system administration and DevOps fundamentals:

### Core Technical Skills
- **Linux System Administration**: User management, service configuration, system policies
- **Storage Management**: LVM concepts, partition encryption, disk management
- **Network Security**: Firewall configuration, SSH hardening, port management
- **Bash Scripting**: System monitoring, text processing, automation
- **Security Best Practices**: Password policies, access control, principle of least privilege
- **Virtualization**: VM setup, snapshots, resource allocation
- **Web Services**: WordPress deployment, lighttpd web server, MariaDB database, PHP configuration

### System Administration Practices
- Understanding the difference between `apt` and `aptitude`
- Configuring AppArmor for Mandatory Access Control
- Managing SELinux policies (for Rocky Linux)
- Reading and interpreting system logs
- Troubleshooting boot issues with encrypted partitions
- Deploying and configuring web services (lighttpd, MariaDB, PHP)
- Managing multiple services and their dependencies

### Problem-Solving Approaches
- Reading official documentation (Debian wiki, man pages)
- Testing configurations in isolated environments
- Understanding security implications of each setting
- Balancing security requirements with system functionality
- Documenting configurations for reproducibility

**Impact**: This project provided foundational knowledge for DevOps and SysAdmin roles, with practical experience in infrastructure security, server hardening, and skills directly applicable to cloud platforms and container technologies.

## 📋 Project Requirements

### Mandatory Constraints
- ✅ VirtualBox (or UTM) virtualization
- ✅ Debian (latest stable) or Rocky Linux
- ✅ No graphical interface (no X.org)
- ✅ Minimum 2 encrypted partitions using LVM
- ✅ SSH service on port 4242 (root login disabled)
- ✅ UFW firewall (only port 4242 open)
- ✅ Hostname: login ending with 42 (modifiable during evaluation)
- ✅ Strong password policy implemented
- ✅ Sudo configured with strict rules
- ✅ User and root accounts with proper group assignments
- ✅ monitoring.sh script running every 10 minutes via cron

### Verification Commands

Below are commands to verify system requirements:

**For Rocky Linux:**
```bash
[root@wil wil]# head -n 2 /etc/os-release
NAME="Rocky Linux"
VERSION="8.7 (Green Obsidian)"

[root@wil wil]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33

[root@wil wil]# ss -tunlp
Netid  State    Recv-Q   Send-Q      Local Address:Port      Peer Address:Port  Process
tcp    LISTEN   0        128             0.0.0.0:4242           0.0.0.0:* users:(("sshd",pid=28429,fd=6))
tcp    LISTEN   0        128                [::]:4242              [::]:* users:(("sshd",pid=28429,fd=4))

[root@wil wil]# firewall-cmd --list-service
ssh

[root@wil wil]# firewall-cmd --list-port
4242/tcp

[root@wil wil]# firewall-cmd --state
running
```

**For Debian:**
```bash
root@wil:~# head -n 2 /etc/os-release
PRETTY_NAME="Debian GNU/Linux 10 (buster)"
NAME="Debian GNU/Linux"

root@wil:/home/wil# /usr/sbin/aa-status
apparmor module is loaded.

root@wil:/home/wil# ss -tunlp
Netid  State    Recv-Q   Send-Q      Local Address:Port      Peer Address:Port
tcp    LISTEN   0        128             0.0.0.0:4242           0.0.0.0:* users:(("sshd",pid=523,fd=3))
tcp    LISTEN   0        128                [::]:4242              [::]:* users:(("sshd",pid=523,fd=4))

root@wil:/home/wil# /usr/sbin/ufw status
Status: active

To                         Action      From
--                         ------      ----
4242                       ALLOW       Anywhere
4242 (v6)                  ALLOW       Anywhere (v6)
```

### Additional Notes
- **Bonus Part**: Implemented
  - Complex partition structure with 7 logical volumes (root, swap, home, var, srv, tmp, var-log)
  - WordPress website with lighttpd, MariaDB, and PHP fully deployed and functional
  - Additional service implemented (e.g., FTP, DNS, or backup service - as per bonus requirements)
  - Firewall rules properly configured for all services
- **Submission**: Only `signature.txt` submitted (VM signature in SHA1)
- **Defense**: Demonstrated live system configuration, created new users, explained all decisions, justified bonus service choice
- **Key Concepts**: Understood differences between apt/aptitude, SELinux/AppArmor, LVM benefits, web server configuration

## ⚖ License

This project is licensed under the MIT License. See the [`LICENSE`](LICENSE) file for more details.

## 📫 Contact

**Berat Boşnak**

- 💼 LinkedIn: [linkedin.com/in/beratbosnak](https://www.linkedin.com/in/beratbosnak)
- 🐙 GitHub: [@beratbosnak](https://github.com/beratbosnak)

---

<div align="center">

*This project is part of the 42 School curriculum - a peer-to-peer learning environment that emphasizes practical skills, problem-solving, and collaboration.*

**42 School** | **Kocaeli Campus** | **2023**

</div>
