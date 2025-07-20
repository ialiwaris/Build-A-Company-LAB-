# KSA Core & Branches VM Setup Guide

This document provides step-by-step instructions to set up the KSA Core and KSA Branches virtual machines (VMs) in Proxmox, including network configuration and essential package installation.

---

## 1. KSA Core VM Setup

### 1.1. Create and Start the VM
- Create a VM in Proxmox named **Ksa_Core**.
- Start the VM and log in.

### 1.2. Network Configuration
- Connect the ISP uplink (e.g., `192.168.100.14/24`) to the VM.
- Test internet connectivity:
  ```sh
  ping 8.8.8.8
  ```
- Connect the following downlinks:
  - `eth1` → KSA JED `eth0` (`10.11.0.1/30`)
  - `eth2` → KSA RUH `eth0` (`10.11.0.5/30`)
  - `eth3` → KSA DMM `eth0` (`10.11.0.9/30`)

### 1.3. Install Required Packages
Update and install essential packages:
```sh
sudo apt update && sudo apt upgrade -y
sudo apt install -y nano git wget iptables-persistent xterm bash-completion frr-pythontools traceroute bpfcc-tools tcpdump vim iputils-ping dnsutils qemu-guest-agent mtr net-tools iperf3 tmux apt-utils jq gpg bridge-utils nmap
```

### 1.4. Automated Setup Script
Create a `setup.sh` file for future use. Example content:

```bash
#!/bin/bash

# Ensure the script is being run as root
if [ "$(id -u)" -ne 0 ]; then
    echo "This script must be run as root!"
    exit 1
fi

echo "Updating package lists..."
apt update -y

echo "Installing required packages..."
apt install -y git nano curl wget vim bash-completion xterm iputils-ping dnsutils qemu-guest-agent mtr net-tools iperf3 tmux tcpdump apt-utils jq gpg bridge-utils traceroute nmap iptables-persistent frr-pythontools

echo "Setting timezone to Asia/Riyadh..."
timedatectl set-timezone Asia/Riyadh

echo "Enabling bash completion..."
echo "source /etc/profile.d/bash_completion.sh" >> ~/.bashrc
source ~/.bashrc

echo "Enabling and starting qemu-guest-agent..."
systemctl enable qemu-guest-agent
systemctl start qemu-guest-agent

echo "Stopping and disabling systemd-resolved..."
systemctl stop systemd-resolved
systemctl disable systemd-resolved

echo "Removing the default /etc/resolv.conf file..."
rm -f /etc/resolv.conf

echo "Creating a new /etc/resolv.conf with nameserver 8.8.8.8..."
echo -e "nameserver 8.8.8.8" > /etc/resolv.conf

# Add custom PS1 and rsz function to both root and srvadmin users
for user in root srvadmin; do
    user_bashrc="/home/$user/.bashrc"
    [ "$user" = "root" ] && user_bashrc="/root/.bashrc"
    cat << 'EOF' >> "$user_bashrc"

export PS1="\[\e[31m\][\[\e[m\]\[\e[38;5;172m\]\u\[\e[m\]@\[\e[38;5;153m\]\h\[\e[m\] \[\e[38;5;214m\]\W\[\e[m\]\[\e[31m\]]\[\e[m\]\\$ "

rsz() {
    if [[ -t 0 && $# -eq 0 ]];then
        local IFS='[;' escape geometry x y
        echo -ne '\e7\e[r\e[999;999H\e[6n\e8'
        read -t 5 -sd R escape geometry || {
            echo unsupported terminal emulator. >&2
            return 1
        }
        x="${geometry##*;}" y="${geometry%%;*}"
        if [[ ${COLUMNS} -eq "${x}" && ${LINES} -eq "${y}" ]];then
            echo "${TERM} ${x}x${y}"
        elif [[ "$x" -gt 0 && "$y" -gt 0 ]];then
            echo "${COLUMNS}x${LINES} -> ${x}x${y}"
            stty cols ${x} rows ${y}
        else
            echo unsupported terminal emulator. >&2
            return 1
        fi
    else
        echo 'Usage: rsz'
    fi
}

EOF
done

echo "Setup complete!"
```

---

## 2. KSA Branches VM Setup

### 2.1. Create and Start the VM
- Create a second VM in Proxmox named **Ksa_branches**.
- Start and log in to the VM.

### 2.2. Install Required Packages and Docker
- If there is no internet, temporarily connect the ISP uplink to install Docker and required packages.
- Make the `setup.sh` file executable and run it:
  ```sh
  sudo chmod +x setup.sh
  sudo ./setup.sh
  ```

### 2.3. Network Configuration
- Connect the three uplinks from the three downlink VMs:
  - `eth0` → from KSA Core `eth1` (`10.11.0.2/30`)
  - `eth1` → from KSA Core `eth2` (`10.11.0.10/30`)
  - `eth2` → from KSA Core `eth3` (`10.11.0.6/30`)

### 2.4. Test LAN Connectivity
Test connectivity:
```sh
ping 10.11.0.9
```

### 2.5. Install and Verify Docker
Install Docker:
```sh
curl https://get.docker.com | sh
sudo docker ps
```

---

## 3. Notes
- Continue with further configuration as needed.
- This document will be updated as the setup progresses.
 

# To Be Continue...
