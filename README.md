
# **Network Topology & Infrastructure Setup Guide**

This guide provides a step-by-step approach to configure network topology and infrastructure. Follow the exercises in sequence to ensure correct setup and persistent configurations across reboots. All tasks must ensure connectivity between devices and Internet access for all containers.

---

## **Table of Contents**
1. [Persistent Configurations](#1-persistent-configurations)  
2. [Exercise 1: IPv4 Addressing & Routing](#2-exercise-1-ipv4-addressing--routing)  
3. [Exercise 2: IPv6 Addressing](#3-exercise-2-ipv6-addressing)  
4. [Exercise 3: Hostname Configuration](#4-exercise-3-hostname-configuration)  
5. [Exercise 4: Internet Connectivity](#5-exercise-4-internet-connectivity)  
6. [Exercise 5: Network Address Translation (NAT)](#6-exercise-5-network-address-translation-nat)  
7. [Exercise 6: Packet Filtering (iptables)](#7-exercise-6-packet-filtering-iptables)  
8. [Exercise 7: SSH Key-based Authentication](#8-exercise-7-ssh-key-based-authentication)  
9. [Exercise 8: Recursive HTTP Search with Authentication](#9-exercise-8-recursive-http-search-with-authentication)  
10. [Exercise 9: File Transfer Between Servers](#10-exercise-9-file-transfer-between-servers)

---

## **1. Persistent Configurations**

- All configurations must persist after reboots.  
- Use `/etc/network/interfaces` and `/etc/network/interfaces.d/*` for network settings.  
- Use the `up` hook for static routes or file modifications (e.g., `/etc/hosts` or `/etc/resolv.conf`).  
- **Important**: Do not modify the `eth0` interface on the host, which is dynamically managed by DHCP.  

---

## **2. Exercise 1: IPv4 Addressing & Routing**

### **IPv4 Addressing**

- Subnet `10.13.$A.0/24` into two subnets:  
  - **VLAN10**: Router0, PC1, PC3  
  - **VLAN20**: Router0, PC2  
- Assign the first usable address to Router0 in each VLAN and sequentially to the PCs.

### **Host – Router Networks**

- **Host - Router0** and **Host - Router-X** network:  
  Subnet `10.11.$B.0/29`, with the host having the first usable IP.

- **Router-X – PC-X Network**:  
  Use `172.18.$C.64/29`, assign the first usable IP to Router-X and the last usable IP to PC-X (before the broadcast address).

### **Routing Configuration**

- Configure default gateways and static routes to enable full communication between stations.  
- **Note**: Do not modify `eth0` on the host to avoid loss of connectivity.

---

## **3. Exercise 2: IPv6 Addressing**

### **VLAN IPv6 Networks**

- Use `2023:E666:$B:$A:$VLANID::/80` for VLAN10 and VLAN20:
  - Assign the first usable address to Router0, followed by addresses for PCs.

### **Host - Router0 IPv6 Connectivity**

- Use `FEC0:5017:$C:$D::/64`:  
  - Assign the first usable address to the host and the second to Router0.

### **IPv6 Routing**

- Ensure IPv6 routing is configured for all devices to communicate via their IPv6 addresses.  
- **Note**: The Router-X – PC-X network does not require IPv6 configuration.

---

## **4. Exercise 3: Hostname Configuration**

### **Hostname Setup**

- Set up the `/etc/hosts` file to allow accessing all devices via hostnames:
  - **Hostnames**: `host`, `Router0`, `PC1`, `PC2`, `PC3`, `Router-X`, `PC-X`
  
### **Persisting Hostnames**

- Save the modified `/etc/hosts` file as `/etc/hosts.orig` and restore it via the `up` hook in `/etc/network/interfaces`.

---

## **5. Exercise 4: Internet Connectivity**

### **NAT Setup**

- Configure NAT on the host to allow containers to reach the Internet.

### **DNS Configuration**

- Create a persistent copy of `/etc/resolv.conf` (`/etc/resolv.conf.orig`) and restore it via the `up` hook to ensure DNS functionality.

---

## **6. Exercise 5: Network Address Translation (NAT)**

### **Port Forwarding**

- Forward the following ports from Router0 to the respective PCs:
  - Port `(14000 + $E)` → SSH to **PC1**  
  - Port `(14000 + $F)` → SSH to **PC2**  
  - Port `(14000 + $G)` → SSH to **PC3**

### **PC-X Tracker**

- Forward traffic from the host on port `(6000 + $K)` to the tracker service on **PC-X**.

---

## **7. Exercise 6: Packet Filtering (iptables)**

### **PC3 Outbound Restrictions**

- Block SMTP and SSH connections initiated by **PC3** to external networks.

### **PC-X Tracker Protection**

- Block access to the PC-X tracker from **PC1**, **PC2**, and **PC3**.

### **PC2 Access Control**

- Block all external connections to **PC2**, except for ICMP and SSH traffic. Use stateful rules to track connection states.

---

## **8. Exercise 7: SSH Key-based Authentication**

### **Passwordless SSH Access**

- Set up passwordless SSH authentication for the `student` user across all systems. Ensure the SSH public keys are distributed correctly.

### **SSH Aliases**

- Configure SSH aliases for easy access:  
  - `ssh pc1` → PC1  
  - `ssh pc2` → PC2  
  - `ssh pc-x` → PC-X  
  - `ssh host` → Host system

---

## **9. Exercise 8: Recursive HTTP Search with Authentication**

### **Script Setup**

- Create the script at `/home/student/scripts/pass` on **PC-X**.

### **Recursive Search & Authentication**

- The script should recursively search through `http://host/.secret/` for `password.txt`.
- Use the following credentials for HTTP authentication:
  - **Username**: `corina`  
  - **Password**: `RL2023`

### **Output**

- Once found, the script should output the contents of `password.txt` only.

---

## **10. Exercise 9: File Transfer Between Servers**

### **Part 1: Script on PC3 (`send-music`)**

1. **Location**: Create the script at `/home/student/scripts/send-music` on **PC3**.
2. **File Selection**: The script will find `.mp3` files with only uppercase letters in the filename (e.g., `SONG.mp3`) in the current directory.
3. **File Upload**: Upload the files to **PC1** at `/home/fs/music` using the credentials:
   - **Username**: `fs`  
   - **Password**: `5$4l0m`

### **Part 2: Script on Host (`copy-firmware`)**

1. **Location**: Create the script at `/root/scripts/copy-firmware` on the **host**.
2. **File Download**: Download `.bin` files from `/opt/my-firmware` on **PC-X** that match the pattern `[a-zA-Z0-9]+\.bin`.
3. **File Upload**: Upload these files to **PC2** at `/tmp/firmware-files/`.
4. **Directory Creation**: Ensure the directory `/tmp/firmware-files/` exists on **PC2**, creating it if necessary.

---

### **Tools and Recommendations**

- Consider using tools like `scp`, `rsync`, `wget`, or `curl` based on the specific task.
- Ensure all scripts are made executable (`chmod +x`) and include the correct shebang (`#!/bin/bash`).
