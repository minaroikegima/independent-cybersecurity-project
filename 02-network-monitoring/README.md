# Network Segmentation & Honeypot Deployment

**Overview**

Built a segmented lab network behind a pfSense firewall and deployed two honeypots (Cowrie and Dionaea) to capture login attempts, command activity, and service probes. A Kali Linux machine on the same network generated simulated attack traffic, which I then matched across packet captures, honeypot logs, and database records. The honeypots are not exposed to the internet, so all traffic described here is simulated from my own lab.

**What Was Built**
- pfSense CE 2.8.1 firewall/router with a NAT WAN interface and an isolated LAN (`10.10.10.0/24`, VirtualBox internal network `LabLAN`)
- DC01 (Windows Server 2019 domain controller from the previous phase) moved behind pfSense with a static address (`10.10.10.10`)
- Honeypot01 (Ubuntu Server) running Cowrie and Dionaea
- Kali Linux (`10.10.10.101`) for scanning and login simulation
- Wireshark for packet capture and analysis

**Network Layout**
- WAN: pfSense reaches the internet through VirtualBox NAT (`10.0.2.15`)
- LAN: pfSense at `10.10.10.1`, handing out DHCP addresses from `10.10.10.100` to `10.10.10.199`
- All lab machines sit on `LabLAN`, so their traffic has to pass through pfSense to reach the internet
- I chose `10.10.10.0/24` instead of pfSense's default `192.168.1.0/24` to avoid overlapping with my home network

**Cowrie (SSH/Telnet honeypot)**
- Installed with pip and initialised with `cowrie init`
- Listens on port 2222, with an iptables rule redirecting port 22 to it
- Presents itself as a Linux server named `FILESERVER01`
- Accepts nearly any credentials by design, then records the login attempt, every command typed, the source IP, and a session ID

**Dionaea (service honeypot)**
- Runs in Docker from the `dinotools/dionaea` image, with logs and data stored in bind mounts on the host
- Emulates services commonly targeted by scanners and malware, including FTP, HTTP/HTTPS, SMB, RPC, MSSQL, and MySQL
- The FTP service presents itself as a Synology DiskStation NAS
- Stores connections, login attempts, and captured files in a SQLite database

**Separating Admin Access From the Honeypot**

The iptables rule that sends port 22 to Cowrie also caught my own admin logins, which landed me in the fake shell instead of the real server. I moved real SSH administration to port 2200, so port 22 now belongs entirely to the honeypot. On Ubuntu this also required stopping and restarting `ssh.socket`, because systemd socket activation controls the listening port separately from `sshd_config`.

**Simulated Attack Activity**
- Ran `nmap -sV` from Kali against Honeypot01 (about 155 seconds) while capturing traffic in Wireshark
- Logged into Cowrie from Kali as a made-up user, then ran `whoami`, `uname -a`, and `cat /etc/passwd`
- Logged into Dionaea's fake FTP service from DC01 with throwaway test credentials

**Findings**
- The same scan is visible from three angles: Wireshark shows one source sending connection attempts to many different ports, Dionaea's database records the connections to its fake RPC and web services (ports 135, 80, and 443), and Cowrie logged the SSH connection and session
- Dionaea's database recorded the FTP login with the username, password, source IP, and timestamp
- Honeypot logs use UTC while Kali showed local time (UTC-4), so timestamps had to be converted before comparing evidence
- The nmap results show how a careful attacker could spot a honeypot: one machine appeared to be a Synology NAS, a Windows file server, a SQL server, and both Debian and Ubuntu at once, and two banners even contained the words "Dionaea honeypot"
- Port 2200, my real SSH service, is reachable by every machine on the LAN and its banner reveals the Ubuntu version

**Challenges and Fixes**
- Admin SSH was being redirected into the honeypot, fixed by moving real SSH to port 2200 and restarting the systemd socket properly
- Cowrie's older install steps no longer matched the current release, so I used the pip-based install and `cowrie init` instead
- Dionaea's FTP and SIP services failed to start because mounting only one sub-folder left its data folders missing, which I found by reading `docker logs` and fixed by mounting `var/lib` and `var/log` as the image documents
- Clipboard paste did not work in the VM console, so I administered Honeypot01 over SSH from DC01

**Possible Improvements**
- Add a pfSense rule so only a management machine can reach port 2200
- Edit Dionaea's service banners so they no longer name the honeypot
- Forward Cowrie and Dionaea logs to a SIEM, which is the next phase

**Screenshots**
- `pfsense-console-interfaces.png`: pfSense console showing the WAN and LAN interfaces
- `dionaea-docker-running.png`: Dionaea container running on Honeypot01
- `dionaea-ftp-banner.png`: Dionaea's fake FTP banner as seen from DC01
- `dionaea-sqlite-login-query.png`: Dionaea database query showing a captured FTP login
- `cowrie-ssh-login-and-commands.png`: Cowrie log showing a login and commands from Kali
- `nmap-service-scan-honeypot01.png`: nmap service scan of Honeypot01 from Kali
- `wireshark-nmap-port-scan-syn-filter.png`: Wireshark view of the scan, filtered to connection attempts from Kali
- `dionaea-nmap-scan-connections.png`: Dionaea database showing the connections recorded during the scan
