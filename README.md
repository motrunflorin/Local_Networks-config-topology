Topology & Infrastructure

In the topology, you have the host routers (the VM hosting the entire infrastructure), Router0 (the container is named R0), and Router-X (under the ID R-X), as well as PC1, PC2, PC3 connected to SW (the switch – there’s nothing to configure on it) through two VLANs (10 and 20) and PC-X (network with R-X).

Note: Due to technical reasons, the device identifiers Router0 and Router-X are shortened to R0 and R-X, respectively. Use the shortened names ONLY WHEN connecting to the container (via the go command or Docker commands if necessary); otherwise, use the full name for everything else (e.g., alias/hosts).
Fun fact: VLAN9 is the network from OpenStack; if running locally, it will be the hypervisor's network (VMWare/VirtualBox) with the physical machine.

You will need to complete the first four exercises in order. These exercises will eventually provide Internet connectivity, and the rest will depend on them.

Persistent Configurations

All configurations must be persistent. They should remain active even after the virtual machine reboots.
To modify specific files in Docker containers (e.g., /etc/hosts, /etc/resolv.conf), you must use an init script or an ifupdown-ng hook. Appropriate references will be provided in the exercises where this is required. For persistent configuration, both the host and containers come pre-installed with ifupdown-ng.

To start configuring, check the existing files at the paths /etc/network/interfaces and /etc/network/interfaces.d/* (⇒ RTFM here ;)).

To persistently configure additional routes (and more), you can use the up hook in the interfaces configuration syntax. Example (fragment):
    iface <intf>
      address A.B.C.D/xx
      up ip route add X.Y.Z.T/yy via Q.W.E.R

The files /etc/network/interfaces* are parsed and executed line by line, and the if[up|down] scripts stop at the first error encountered. For instance, if you have an invalid route in the iface section, the rest of the declarations will not be applied.

You can use the following one-liner to quickly check an interface:
    ifdown --force <intf>; ifup <intf>; ip a sh; ip ro sh

Hint: Use ifdown and ifup with the -a parameter to start ALL declared interfaces!
However: DO NOT RUN ifdown -a on the host! You will lose connectivity on eth0!!!

To save iptables rules, follow the steps here: https://www.serveracademy.com/courses/linux-fundamentals/how-to-save-iptables-rules-permanently/ (there are several methods, e.g., you can add up hooks to an interface). UNDER NO CIRCUMSTANCES should you install the packages described in tutorials (especially ifupdown – it will break your VM; you already have ifupdown-ng!!).

f using multiple files in scripts (e.g., calling a script from another script), always use absolute paths. For example, use /root/scripts/make-juju.py instead of ./make-juju.py to avoid relying on the current working directory.
DON’T forget to make them executable and include the shebang!

Exercise 1 IPv4 Addressing + Routing
Configure the IPv4 addresses for all links in the topology as follows:

Subnet the space 10.13.$A.0/24 into two fixed-size networks; assign the first subnet to VLAN10 and the second to VLAN20. Assign the first usable address to Router0, and then assign addresses to the stations in order (i.e., Router0, PC1, PC3 for VLAN10, and Router0, PC2 for VLAN20). Note: The VLANs are already declared on the router in the file located at /etc/network/interfaces.d/.

For the host – Router0 and host – Router-X networks, optimally subnet the space 10.11.$B.0/29 and assign them in the order mentioned. The host must always have the first usable address.

For the Router-X – PC-X network: use 172.18.$C.64/29. The router will have the first usable address, and PC-X will use the last usable address (note: not the broadcast address, but the one before it!).

Recommendation: Do not modify anything on the SW container (switch), or you risk breaking your infrastructure!

Configure IPv4 routing (both default gateways and static routes) so that all stations can access each other via IP addresses.
Warning: The IP address on the host’s eth0 interface is dynamically assigned by the hypervisor via DHCP. DO NOT touch this interface (never go full ifdown!), or you risk losing access to the VM!

Exercise 2  IPv6 Addressing
Configure IPv6 addresses for the VLAN10 and VLAN20 networks. (Note: the $VLANID variable will have values 10 and 20 respectively):

Use the space 2023:E666:$B:$A:$VLANID::/80.
Same order as above (like for IPv4): Router0 will get the first usable address, then the PCs will be numbered.
Configure IPv6 connectivity between Router0 and the host:

Use the space FEC0:5017:$C:$D::/64.
The first usable address goes to the host, and the second goes to Router0.
Configure IPv6 routing to allow communication between all systems using IPv6 addresses.

The Router-X – PC-X network will NOT have IPv6 addresses.

Exercise 3  Hosts
Make the necessary configurations so that devices can be accessed by their hostnames (use names like host, Router0, PC1, PC2, PC3, Router-X, PC-X – be mindful of uppercase letters!). Add entries only for IPv4 addresses.

The /etc/hosts file in the containers is special (mounted as a --bind volume by Docker). This causes any changes to be lost after every container restart (e.g., VM reboot or restarting the rl-topology service).
If you want these changes to persist, save the file to another path (e.g., /etc/hosts.orig) and restore it every time the container starts using an up hook in /etc/network/interfaces.
Oh, and don’t use cp; use cat + bash redirection (since it’s a volume, you can’t delete + recreate the file – you need to truncate and overwrite), e.g.:
    iface <intf>
      up cat /etc/hosts.orig >/etc/hosts

Exercise 4  Internet Connectivity
Configure the necessary settings so that all four containers have Internet access:

Configure NAT on the host system so that containers can receive responses from the Internet.
Set up the four containers to access Internet resources by domain name (DNS).
Note: Do not translate any packets routed between containers (e.g., they should see each other’s original IPs).

The resolv.conf file is managed as a volume by Docker :( … same story as in the previous exercise ⇒ same solution (e.g., create a /etc/resolv.conf.orig file and overwrite /etc/resolv.conf with an up hook in the interfaces file).

Exercise 5  Network Address Translation
Configure NAT rules on the host and/or Router0 (whichever is appropriate):

Connections to Router0 on ports (14000 + $E), (14000 + $F), and (14000 + $G) should lead to SSH connections to PC1, PC2, and PC3, respectively.
Connecting to the host on port (6000 + $K) should lead to connecting to the tracker on PC-X.
Tip: Be careful how you test this: DNAT will ONLY work if you come from a network external to the router.

Exercise 6  Packet Filtering (iptables)
Set up packet filtering on the host and/or Router0 (depending on the task) as follows:

Block SMTP and SSH connections initiated by PC3 outside its network (including to other routers!).
Block connections to the tracker running on PC-X from PC1, PC2, and PC3.
Block ALL external connections (i.e., from IPs outside the station) to PC2, except for ICMP and SSH protocols.
Note: Do not block connections initiated by PC2 or their responses! Use stateful rules (i.e., connection tracking).

Exercise 7 SSH Keys
Configure the SSH service on PC1, PC2, PC-X, and the host to allow passwordless authentication using public keys for the user student across all systems (as student).

Use SSH aliases to simplify commands for connecting between the systems:
ssh pc1 should connect to PC1 as the student user.
ssh pc2 should connect to PC2 as the student user.
ssh pc-x should connect to PC-X as the student user.
ssh host should connect to the host as the student user.
Note: ssh-copy-id uses password-based SSH authentication to copy the key. OpenStack VMs do not allow such authentication on the host, so you must copy the keys manually (or better yet, use Ansible for automation).

Exercise 8  – Recursive Search on HTTP Server with Authentication
Create a script on the PC-X system at /home/student/scripts/pass that performs the following tasks:

Recursive Search: The script should search through all the pages under http://host/.secret/ and locate the file password.txt (e.g., it may be found at http://host/.secret/sub/path/password.txt).
Authentication: You will need to authenticate using HTTP with the following credentials:
Username: corina
Password: RL2023
Output: Once the password.txt file is located, the script should output only its contents.

Ex. 9 Download / Upload Files Between Servers
Part 1: Script on PC3 (send-music)
Create a script on PC3 at /home/student/scripts/send-music that will do the following:

File Selection: The script will search for and select only .mp3 files that contain only uppercase letters in the file name (e.g., SONG.mp3) from the current directory ($PWD).
Upload: The script will upload these files to PC1, in the destination /home/fs/music.
Credentials: The following credentials will be used for uploading:
Username: fs
Password: 5$4l0m
Part 2: Script on the host (copy-firmware)
Create a script on the host at /root/scripts/copy-firmware, which will:

Download Files: The script will download only files from the directory /opt/my-firmware on PC-X that match the pattern [a-zA-Z0-9]+\.bin (alphanumeric characters followed by the .bin extension).
Upload Files: The downloaded files will be uploaded to PC2, in the directory /tmp/firmware-files/.
Create Directory: If the directory /tmp/firmware-files/ does not exist on PC2, the script should create it.
Recommended Tools:
You can use tools like ftp, wget, curl, scp, or rsync, depending on the scenario. Study each tool before deciding which one to use.
Additional Notes:
You are dealing only with files (non-recursive transfer), and you should not copy files into subdirectories. The files must be placed directly in the specified path; otherwise, they will not be detected by the checker (and the task will be marked as incorrect).
The pattern used in the description is PCRE (Perl-Compatible Regular Expressions), but this format is not necessarily implemented in the tools used! Check the documentation or man page of the tool you're using (pay attention to glob vs regex – they have major syntax differences).
For testing, you can either create the necessary files in a temporary directory or let the checker do this automatically (in a temporary directory, which you will need to locate, though it should be clear where it is placed).
The checker for this task will refuse to run if the scripts do not exist or are not executable, displaying the error "skipped."
