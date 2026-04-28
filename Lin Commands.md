## Tips
***

### Close Ports

Ports on Linux can be closed by stopping the process that is running on that port. To see what ports are open, use the `ss -tulpn` command. If there is no `pid` attached to the open port and neither is the service listed, use the `sudo lsof -i :$PORT` to get a PID for that process. **Do not forget the sudo command.**

## Tools
***
### enum4linux

```bash
enum4linux [options] ip
```

| tag | function                                    |
| --- | ------------------------------------------- |
| -U  | get Userlist                                |
| -M  | get machine list                            |
| -N  | get namelist dump (different from -U and-M) |
| -S  | get share list                              |
| -P  | get password policy information             |
| -G  | get group and member list                   |
| -a  | all of the above (full basic enumeration)   |
| -o  | os info                                     |
| -r  | RID cycling                                 |
### SMBClient

- `Syntax: smbclient -L //IP/ -N`: get share list
- `smbclient //[IP]/[SHARE] -U [USERNAME] -p [PORT]`

```bash
smbclient //10.10.10.10/secrets -U Anonymous -p 445
```

**SMBClient Commands**
- Once inside the share, you can view the available commands by typing `help`. The most useful of which are:

| command    | desc                                |
| ---------- | ----------------------------------- |
| ls or dir  | List files and directories          |
| cd [DIR]   | Move to a different directory       |
| get [FILE] | Download the file to your AttackBox |
### Hydra

- Hydra is a very fast online password cracking tool, which can perform rapid dictionary attacks against more than 50 Protocols, including Telnet, RDP, SSH, FTP, HTTP, HTTPS, SMB, several databases and much more.

```bash
hydra -t 4 -l dale -P /usr/share/wordlists/rockyou.txt -vV 10.10.10.6 ftp
```

| Section                 | Function                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------ |
| hydra                   | Runs the hydra tool                                                                  |
| -t 4                    | Number of parallel connections per target                                            |
| -l [user]               | Points to the user who's account you're trying to compromise                         |
| -P [path to dictionary] | Points to the file containing the list of possible passwords                         |
| -vV                     | Sets verbose mode to very verbose, shows the login+pass combination for each attempt |
| IP machine              | The IP address of the target machine                                                 |
| ftp / protocol          | Sets the protocol                                                                    |
### mount

```bash
# get path
locate showmount
which showmount
```

```bash
# list the NFS shares
/usr/sbin/showmount -e [IP]
```

```bash 
sudo mount -t nfs IP:/share /tmp/mount/ -nolock
```


| Tag       | Function                                                                     |
| --------- | ---------------------------------------------------------------------------- |
| -mount    | Execute the mount command                                                    |
| -t nfs    | Type of device to mount, then specifying that it's NFS                       |
| IP:/share | The IP Address of the NFS server, and the name of the share we wish to mount |
| -nolock   | Specifies not to use NLM locking                                             |
### John

```bash
# john misc
unshadow /etc/passwd /etc/shadow > output.db
john --wordlist=/usr/share/wordlists/rockyou.txt output.db
```
***
### Gobuster 

```bash
gobuster dir -u 'http://192.158.45.222/' -w /usr/share/wordlists/dirb/common.txt
```

###  Feroxbuster
```bash

```
## Commands
***

### download it onto our attacking machine

```bash
# copy file bash form victim to attack
scp -i key_name username@MACHINE_IP:/bin/bash ~/Downloads/bash

# copy files from one computer to another
scp -r FILE_CP username@TARGET_IP:/path
# path: path on target
# FILE_CP: file to copy
```

### Find Magic

```bash
find / -perm /4000 -type f 2>/dev/null # suid binaries

find / -perm -type f -u=s 2>/dev/null
```


### Start an SMB server on Kali

```bash
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py tools .
```

### Listen

```bash
ss -lntp
# OR
netstat -lntp 
# kill listen
sudo kill 7608

```

### Upgrade Shell

```bash
# Spawn PTY
# python
python -c 'import pty; pty.spawn("/bin/bash")'

python3 -c 'import pty; pty.spawn("/bin/bash")'

#perl
perl -e 'exec "/bin/bash";'

# ruby
ruby -e 'exec "/bin/bash"'

# script /bin/bash
script /dev/null -qc /bin/bash

```

```bash
ssh-keygen -f "/home/kali/.ssh/known_hosts" -R "IP"
```