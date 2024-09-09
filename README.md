Topology & Infrastructure Configuration Instructions
This document outlines the configuration steps for setting up the network infrastructure and addressing in the virtual topology. Follow the exercises in order to ensure proper connectivity and functionality. All configurations must be persistent and survive a VM reboot.

Table of Contents
Persistent Configurations
Exercise 1: IPv4 Addressing & Routing
Exercise 2: IPv6 Addressing
Exercise 3: Hostname Configuration
Exercise 4: Internet Connectivity
Exercise 5: Network Address Translation (NAT)
Exercise 6: Packet Filtering (iptables)
Exercise 7: SSH Key-based Authentication
Exercise 8: Recursive HTTP Search with Authentication
Exercise 9: File Transfer Between Servers
Persistent Configurations
Configurations must be persistent across reboots.
Use /etc/network/interfaces and /etc/network/interfaces.d/* for network settings.
Use the up hook for additional commands like static routes or file modifications (e.g., /etc/hosts or /etc/resolv.conf).
Important: Do not modify eth0 on the host as it is managed dynamically via DHCP.
Exercise 1: IPv4 Addressing & Routing
Subnet the 10.13.$A.0/24 network:

Assign the first subnet to VLAN10 (Router0, PC1, PC3).
Assign the second subnet to VLAN20 (Router0, PC2).
Host - Router0 and Host - Router-X network:

Subnet the 10.11.$B.0/29 space and assign the first usable IP to the host.
Router-X - PC-X Network:

Use the subnet 172.18.$C.64/29, with Router-X using the first usable address and PC-X the last.
Routing:

Configure IPv4 routing and default gateways so all systems can communicate.
Do NOT modify the eth0 interface on the host.
Exercise 2: IPv6 Addressing
VLAN Networks:

Use the range 2023:E666:$B:$A:$VLANID::/80 for VLAN10 and VLAN20.
Assign the first usable address to Router0 and the subsequent ones to the PCs.
Router0 - Host IPv6 Network:

Use the space FEC0:5017:$C:$D::/64, with the host receiving the first usable address.
Routing:

Enable IPv6 routing to allow communication between all devices using IPv6 addresses.
Exercise 3: Hostname Configuration
Hostname Access:
Configure /etc/hosts files so that all devices can be accessed by hostname (Router0, PC1, PC2, PC3, Router-X, PC-X, host).
Persisting /etc/hosts:
Save /etc/hosts as /etc/hosts.orig and restore it at boot via the up hook.
Exercise 4: Internet Connectivity
NAT Configuration:

Configure NAT on the host to allow the containers to access the Internet.
DNS Configuration:

Override /etc/resolv.conf with a persistent copy (/etc/resolv.conf.orig) using the up hook.
Exercise 5: Network Address Translation (NAT)
Port Forwarding:

Forward connections to Router0 on ports (14000 + $E), (14000 + $F), and (14000 + $G) to the SSH ports of PC1, PC2, and PC3, respectively.
PC-X Tracker:

Forward connections on the host on port (6000 + $K) to the tracker on PC-X.
Exercise 6: Packet Filtering (iptables)
PC3 Outbound Restrictions:

Block SMTP and SSH connections initiated by PC3 to any external network.
PC-X Tracker Protection:

Block access to the PC-X tracker from PC1, PC2, and PC3.
PC2 Access Control:

Block all external connections to PC2 except for ICMP and SSH.
Exercise 7: SSH Key-based Authentication
Passwordless Authentication:
Configure SSH for passwordless login using public keys for the user student across all devices.
SSH Aliases:
Set up SSH aliases (ssh pc1, ssh pc2, etc.) for ease of use.
Exercise 8: Recursive HTTP Search with Authentication
Script Location:

Create the script at /home/student/scripts/pass on PC-X.
Recursive Search:

The script should recursively search through the directory http://host/.secret/ to locate the file password.txt.
Authentication:

Use HTTP authentication with the credentials:
Username: corina
Password: RL2023
Output:

Once found, the script should print only the contents of the password.txt file.
Exercise 9: File Transfer Between Servers
Part 1: Script on PC3 (send-music)
Script Location:

Create the script at /home/student/scripts/send-music on PC3.
File Selection:

Select .mp3 files that consist of only uppercase letters in the file name from the current directory.
Upload:

Upload the files to PC1 at /home/fs/music using the following credentials:
Username: fs
Password: 5$4l0m
Part 2: Script on the Host (copy-firmware)
Script Location:

Create the script at /root/scripts/copy-firmware on the host.
Download Files:

Download files from /opt/my-firmware on PC-X that match the pattern [a-zA-Z0-9]+\.bin.
Upload:

Upload the files to PC2 at /tmp/firmware-files/.
Create Directory:

Ensure the directory /tmp/firmware-files/ exists on PC2, creating it if necessary.
Notes:
Use appropriate tools like scp, rsync, wget, or curl for file transfer, depending on the situation.
Ensure all scripts are executable and include the proper shebang (#!/bin/bash).
