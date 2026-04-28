#  1. SMB 
## 1. 1 ENTRY POINT ENUMERATION

### Scan smb-os-discovery,smb-protocols

```bash
# smb-os-discovery,smb-protocols
sudo nmap -sV -p139,445 --script smb-os-discovery,smb-protocols 192.168.1.9
Starting Nmap 7.98 ( https://nmap.org ) at 2025-12-23 09:13 -0500
Nmap scan report for 192.168.1.9
Host is up (0.065s latency).

PORT    STATE    SERVICE     VERSION
139/tcp filtered netbios-ssn
445/tcp open     netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
MAC Address: 3C:F8:62:5C:62:A5 (Intel Corporate)
Service Info: Host: UBUNTU

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: ubuntu
|   NetBIOS computer name: UBUNTU\x00
|   Domain name: \x00
|   FQDN: ubuntu
|_  System time: 2025-12-23T14:14:05+00:00
| smb-protocols: 
|   dialects: 
|     NT LM 0.12 (SMBv1) [dangerous, but default]
|     2.0.2
|     2.1
|     3.0
|     3.0.2
|_    3.1.1

```

- Port 445
```bash
445/tcp open     netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)

- 445 open -> SMB direct hosting
- Samba smbd 4.3.11 -> linux Samba
- WORKGROUP ->  Không join domain AD
  
--> Đây là standalone Linux file server, không phải DC / member server.
```

- smb-os-discovery
```bash

OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
Computer name: ubuntu
NetBIOS computer name: UBUNTU

- Samba giả lập SMB stack của Windows**
- Windows 6.1 = Windows 7 / Server 2008R2
- Để client Windows tương thích

--> OS thật là Linux, chỉ là SMB fingerprint giả.

```

- smb-protocols
```bash

dialects:
  NT LM 0.12 (SMBv1) [dangerous, but default]
  2.0.2
  2.1
  3.0
  3.0.2
  3.1.1
  
- NT LM 0.12 (SMBv1) -> Rất xấu
- SMBv2+ -> OKI

-->  Samba đang bật SMBv1 --> misconfiguration
```

> [!note]
> SMBv1 cho phép:
> 	•	Downgrade attack
> 	•	NTLM relay dễ hơn
> 	•	Enumeration lỏng hơn
> 	•	Legacy auth
> 
> --> Đây là pivot condition, không phải exploit trực tiếp

### Scan smb-security-mode
```bash

sudo nmap --script smb-security-mode -p445 -Pn -sT  192.168.1.9 
Starting Nmap 7.98 ( https://nmap.org ) at 2025-12-23 09:18 -0500
Nmap scan report for 192.168.1.9
Host is up (0.11s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)


```

- smb-security-mode
	- <font color="#ffff00">account_used: guest</font>
		- Nmap đăng nhập SMB bằng guest
		- Server CHO PHÉP guest session
		- Không cần username/password
		--> <font color="#de7802">Anonymous / Guest SMB access đang bật</font>
	- <font color="#ffff00">authentication_level: user</font>
		- SMB yêu cầu user-level auth
		-  guest vẫn được map thành user
	- <font color="#ffff00">challenge_response: supported</font>
		- SMB hỗ trợ challenge–response
		- Tức là NTLM
	- <font color="#ffff00">message_signing: disabled (dangerous, but default)</font>
		- SMB signing đảm bảo <font color="#de7802">toàn vẹn & chống MITM</font>
		- Nếu **disabled**:
		    - SMB <font color="#92d050">không phát hiện bị relay</font>
		    - Có thể <font color="#92d050">MITM / NTLM relay</font>

## 1.2  ATTACK PATH
###  Deep Enumeration 
```bash

smbclient -L //192.168.1.9 -N
smbmap -H 192.168.1.9
enum4linux -a 192.168.1.9
  ```