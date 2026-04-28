***
```bash
Kali  →  Ubuntu (pivot)  →  Internal Service

┌──────────────┐        172.10.10.0/24         ┌────────────────┐
│     KALI     │ ─────────────────────────▶    │     UBUNTU     │
│ 172.10.10.140│                               │ 172.10.10.134  │
└──────────────┘                               └───────┬────────┘
                                                       │
                         ┌─────────────────────────────┼─────────────────────────────┐
                         │                             │                             │
                172.16.10.0/24                   10.1.0.0/24                    (Other nets...)
                         │                             │
            ┌────────────▼────────────┐    ┌──────────▼──────────┐
            │        WEB SERVER       │    │     MYSQL SERVER    │
            │    172.16.10.12 : 80    │    │   10.1.0.16 : 3306  │
            └─────────────────────────┘    └─────────────────────┘

```

reference: https://book.hacktricks.wiki/en/generic-hacking/tunneling-and-port-forwarding.html
## Summary All Commands

```bash

# Port forward using Socat
socat -ddd TCP-LISTEN:$NEW_PORT,fork,reuseaddr TCP:$INTERNAL_IP:$INTERNAL_IP_PORT
socat -ddd TCP-LISTEN:2345,fork,reuseaddr TCP:10.1.0.16:3306

# SSH Local Port Forward
ssh -N -f -L  [$BIND_ADDRESS:]$LOCAL_PORT:$MOST_INTERNAL_IP:$MOST_INTERNAL_PORT $middle_user@$middle_system
ssh -N -f -L 0.0.0.0:8080:172.16.10.12:80 pentest-lab@172.10.10.134

# SSH dynamic - proxychains
ssh -N -D 0.0.0.0:$OPEN_NEW_PORT $username@$MIDDLE_IP
ssh -N -D 0.0.0.0:9999 pentest-lab@172.10.10.134
# add "socks5 172.0.0.1 9999" to proxychains.conf (kali IP)
# use internal machine's IP as target IP directly. (172.16.XXX.XXX)
sudo vim /etc/proxychains4.conf

# SSH remote port forward
ssh -N -R 127.0.0.1:5555:172.16.10.12:80 kali@172.10.10.140 -v
# run commands with 127.0.0.1 as targetIP

# SSH remote dynamic forwarding
ssh -N -R 9998 $kali_user@$attack_IP -v
# add "socks5 127.0.0.1 9998" to proxychains.conf.

# sshuttle
# syntax
sshuttle -r $user@$ip_address:$port 
# here is where the magic happens. You can add any other subnets that the internal machine might have access to.
# IPs from left to right - 172.10.10.134, 172.16.10.0/24, 10.1.0.0/24
sshuttle -r pentest-lab@172.10.10.134 172.16.10.0/24 10.1.0.0/24
```

***

# Port Forwarding & SSH Tunneling
## Port Forwarding
### 1. 1 Port Forwarding  using Socat
1. Creating a forward on the Ubuntu Machine with SOCAT:

```bash
socat TCP-LISTEN:2345,fork,reuseaddr tcp:172.16.10.12:80 &
socat TCP-LISTEN:1234,fork,reuseaddr tcp:10.1.0.16:3306 &

# start a socat process on the dual_homed machine
# SOCAT WORKING
# socat "LISTEN" and then "FORWARD"
# socat TCP-LISTEN:$PORT,fork,reuseaddr -- opens a port on the DH machine.
# fork -- creates a new process.
# reuseaddr -- reuse port
# TCP:$INTERNAL_TARGET_TP:$PORT -- this port should already be open.
# Ports 0-1024 require privs, do not use.
```

- Check handshake by netcat
```bash 
nc -vz 172.10.10.134 1234
```

![[Pasted image 20260123150951.png]]
- Check Listen Socat
```bash
ss -lntp | grep socat
# OR
netstat -lntp | grep socat
```

![[Pasted image 20260123151359.png]]

2. Interacting with a port on one of the internal machines (Mysql):
```bash
# now just interact with the internal IP just like normal.
mysql -h $UBUNTU_IP -P $INTERNAL_PORT -u $USER -p --skip-ssl 
mysql -h 172.10.10.134 -P 1234 -u root -p --skip-ssl

# $UBUNTU_IP -- send all traffic to the Ubuntu machine.
# Ubuntu Machine will forward it to the internal machine (via socat),
```

3. Reverse  Port Forwarding SOCAT
- On Kali Linux:
```bash
# 
socat TCP-LISTEN:9090,reuseaddr,fork TCP-LISTEN:8080,reuseaddr,fork

# Port 9090: Get connection from Ubuntu
# Port 8080: Use (curl/browser) for the attacker
```

- On ubuntu:
```bash
socat TCP:172.10.10.140:9090 TCP:172.16.10.12:80
```

- On Kali Linux:
```bash
# test 
curl http://127.0.0.1:8080
```
***
## 2. Tunnelling
### 2.1 SSH Local Port Forwarding

- Scan internal ports via SSH tunnel.

```bash

IP_INTERNAL=127.0.0.1   # local end của tunnel
for i in $(seq 1 1024); do

    nc -z -w 1 $IP_INTERNAL $i 2>/dev/null && echo "Port $i OPEN"

done
```

-  It is the method used in SSH to forward the ports of application from a client machine to the server machine. By making use of this, the SSH client listens for connections on a port which has been configured, and tunnels to an SSH server when a connection is received. This is how the server connects to a destination port which is configured and is present on a machine other than the SSH server
- This opens a connection to the machine with IP 172.16.10.12 and forwards any connection of port 80 on the local machine to port 8080

```bash

ssh -N -L -f [$BIND_ADDRESS:]$LOCAL_PORT:$MOST_INTERNAL_IP:$MOST_INTERNAL_PORT $middle_user@$middle_system
# the three networks in this case from left to right are
# > 172.10.10.140
# > 10.1.0.16
# > 172.10.10.134
# N -- No command - is to not open a shell instead just start a process
# L -- Local Port Forwarding
# -L [BIND_ADDRESS:]LOCAL_PORT:MOST_INTERNAL_IP:MOST_INTERNAL_PORT
	# BIND_ADDRESS -- (127.0.0.1 or 0.0.0.0): 0.0.0.0 --> Bind ports on ALL interfaces of the local machine, Not just 127.0.0.1 
	# LOCAL_PORT -- The port is open on the local machine.
	# MOST_INTERNAL_IP -- IP deep internal - Only middle_chapter is visible.
	# MOST_INTERNAL_PORT -- Internal Port Service
# f -- Fork to background - Get SSH running in the background.
# above command can be re-written as:
ssh -N -L -f 8080:172.16.10.12:80 pentest-lab@172.10.10.134
```
***



### 2.2 SSH Dynamic Port Forwarding (Proxychains)
-  Same setup as above, however, we don't always know what systems are online & what port they are listening on. (Practically speaking)
- We want to be able to scan any network through `nmap` from our Kali Machine.
- From `kali` we will simply open a new port (`port 9999` in this case) & bind it to `ubuntu`. This will send any command from our system, directly to `ubuntu`
-  We just need to edit the `/etc/proxychains4.conf` file in our kali and setup a `socks5 proxy`. To do this, enter the IP of `ubuntu` and the port we just opened. Meaning, this line `socks5 $IP_KALI $PORT`. In this case, we will add the following line - `socks5  127.0.0.1 9999`.

```bash
# open a new port on kali and foward all packets to ubuntu:
ssh -N -D 0.0.0.0:9999 pentest-lab@172.10.10.134
# D -- Dynamic, You can go anywhere you want (DNS, web, nmap TCP connect, etc.)

# Running proxychains:
proxychains4 [any command]
proxychains4 mysql -h 10.1.0.16 -P 3306 -u root -p --skip-ssl
proxychains4 nmap -v -sT -p- -T5 -Pn -n 10.1.0.16 
# -n to make sure no DNS errors occur.
# Do not use -sS(because SYN scans don't go through SOCKS)


```



![[Pasted image 20260123215102.png]]
***

### 2.3  SSH Remote Port Forwarding

First of all make sure that in the `/etc/ssh/sshd_config` file, you had added the line `PasswordAuthentication yes`. This will allow any machine to connect back to Kali with the user creds. Components:
- - In this case there is a firewall that only allows inbound packets on `port 8090` because of the web server running on `ubuntu`.
- So an outboud connection has to be made (bind shell of sorts) from `ubuntu` to `kali` attack machine. This can be achieved by running the SSH command on `ubuntu` that will open a new port on the loopback address of the `Kali` machine & bind it to an open port (`port 80` in this case) on the internal machine.
- Then from the attak machine, the internal victim can be interacted with, by specifying the loopback address as the target IP address in all commands.
- 
```bash
# on the Ubuntu server
ssh -N -R 127.0.0.1:$OPEN_NEW_PORT:$INTERNAL_IP:$PORT $kali_user@$kali_IP -v
ssh -N -R 127.0.0.1:8000:172.16.10.12:80 kali@172.10.10.140 -v
```

***
### 2.4  SSH Remote Dynamic Port Forwarding

```bash
# on victim - Ubuntu 
ssh -N -R 9998 $kali_user@$attack_IP -v

# on kali add - socks5 127.0.0.1 9998 to /etc/proxychains4.conf

proxychains4 nmap -sT -vvv -p $any_port 10.1.0.16 -T5 -sCV # use INTERNAL IP of multiserver.
```

***
## 3. Using Sshuttle
When nothing works, use shuttle. There are certain caveats to using this method:

1. Requires root privs on the SSH Client
2. Python3 on SSH Server In this module, sometimes Kali was the SSH Server (when the victim connected to the attacker), and other times - both - SSH Client & SSH Server - were victims.
- Just run socat and open a new port on the victim machine and also fork the process so that it creates a new process in the background using socat's syntax - `socat TCP-LISTEN:2222,fork TCP:$INTERNAL_IP:$PORT`.
- On the attack machine, use sshuttle to connect to the dual-homed machine on the newly opened port with socat (which will forward everything to the internal machine).

```bash
# syntax
sshuttle -r $user@$ip_address:$port 

# here is where the magic happens. You can add any other subnets that the internal machine might have access to.
# IPs from left to right - 172.10.10.134, 172.16.10.0/24, 10.1.0.0/24
sshuttle -r pentest-lab@172.10.10.134 172.16.10.0/24 10.1.0.0/24

# Interacting with a port on one of the internal machines (Mysql)
mysql -h 10.1.0.16 -P 3306 -u root -p --skip-ssl

```

![[Pasted image 20260124081336.png]]

***
## 4. Using Plink 
- `plink` is the CLI version of the PuTTY client for windows. It has most of the features SSH has but one - Remote Dynamic Port Forwarding.
### 4.1 Local SSH Tunnelling using Plink.exe

```bash
# On Windows
plink.exe -N -L 5555:$INTERNAL_IP:$PORT $kali_user@$attack_IP
# D 5555 -- open SOCKS5 tunnel
# N -- no command

```

***

## 5.  Using ssh.exe
- Windows 10 - `Version 1803 && Version 1709` - and later - come with the OpenSSH Suite of tools (`ssh-*`) installed by default in the `%systemdrive%\Windows\System32\OpenSSH\` directory. Use the `where ssh` comand on windows to locate the SSH binary on the windows victim. Components: There is no magic happening here. Once we know that the Windows system has SSH, we can use any method from above to set up port forwarding on the windows system.

```bash
C:\Users\rdp_admin> ssh -N -D $New_Port hutgrabber@kali_IP # dynamic remote port forward.
```

- After this, add the port to proxychains with your own loopback address as shown in the Dynamic Remote PF techniques.