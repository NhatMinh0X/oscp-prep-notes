# 1. Access Control and Rootly Powers
## 1.1 Standard UNIX access control 
- he scheme follows a few basic rules:
	- Access control decisions depend on which user is attempting to perform an operation, or in some cases, on that user’s membership in a UNIX group. 
	-  Objects (e.g., files and processes) have owners. Owners have broad (but not necessarily unrestricted) control over their objects. 
	- You own the objects you create. 
	-  The special user account called “root” can act as the owner of any object. 
	- Only root can perform certain sensitive administrative operations.
### 1.1.1 Filesystem access control
- link book: <font color="#ffff00">UNIX and Linux System Administration Handbook(66-67)</font>
- <font color="#de7802">groups</font>: /etc/group
- these days group information is often stored in a network database system such as <font color="#ffff00">LDAP</font>; see Chapter 17, Single SignOn, for details.
- You can determine the ownerships of a file with <font color="#de7802">ls -l</font>:
```bash
ls -l ~garth/todo 
-rw-r----- 1 garth staff 1259 May 29 19:55 /Users/garth/todo

# This file is owned by user garth and group staf.
# the codes mean that garth can read or write the file and that members of the staff group can read it.

# user identification numbers (UIDs for short) are mapped to usernames in the /etc/passwd file
# group identification numbers (GIDs) are mapped to group names in /etc/group
```
### 1.1.2 Process ownership
- 1. Real UID -> Who started the program?
- 2. Effective UID -> actual rights being used
- 3. Saved UID → retain high authority for reuse
### 1.1.3 The root account
- The root account is UNIX’s omnipotent administrative user. It’s also known as the superuser account, although the actual username is “root”.
```bash
UID = 0 
```
- MSome examples of restricted operations are:
	- Creating device files 
	- Setting the system clock 
	- Raising resource usage limits and process priorities 
	- Setting the system’s hostname 
	- Configuring network interfaces 
	- Opening privileged network ports (those numbered below 1,024) 
	- Shutting down the system
### 1.1.4 setuid and setgid execution
-
## 1.2 Management of the root account
### 1.2.1 Root account login
- To begin with, <font color="#ffff00">root logins leave no record of what operations were performed as root.</font> That’s bad enough when you realize that you broke something last night at 3:00 a.m. and can’t remember what you changed; it’s even worse when an access was unauthorized and you are trying to figure out what an intruder has done to your system. <font color="#ffff00">Another disadvantage is that the log-in-as-root scenario leaves no record of who was actually doing the work</font>. If several people have access to the root account, <font color="#ffff00">you won’t be able to tell who used it and when</font>.
- 
```bash
su - username
  # access that person’s account
```

- On some systems, the root password allows an  ` su ` or login to any account. root can `su` to any account without entering a password
- We consider `su` to have been largely superseded by `sudo`
- <font color="#ffff00">Sudo</font>:
	- `sudo` takes as its argument a command line to be executed as `root`
	- `sudo` consults the file <font color="#ffff00">/etc/sudoers</font> (<font color="#ffff00">/usr/local/etc/sudoers</font> on FreeBSD)

# 2. Permissions
## 2.1  File Permissions
### 2.1.1 How does the kernel view permissions?
- Kernel **does not care about username**
- Kernel only cares about:
    - UID (user id)
    - GID (group id)
    - **permission bits (rwx)**
> Permissions **do not describe what the file does**, but rather describe: **what the kernel allows the process to do with that file**.

```bash
$ ls -l Desktop/ 
drwxr-xr-x 2 pete penguins 4096 Dec 1 11:45 .
```

- <font color="#ffff00">Permission structure: </font> `drwxr-xr-x`
	- `d`: Loại file (dỉectory)
	- `rwx`: quyền owner
	- `r-x`: quyền group
	- `r-x`: quyền other
--> <font color="#ffff00">**The last 9 bits are the most important.**</font> 
### 2.1.2 Permissions on a file (regular file)

| Permission | What does the file mean?             | Numerical value |
| ---------- | ------------------------------------ | --------------- |
| r          | read file contents (`cat`, `less`)\| | 4               |
| w          | wite/edit contents                   | 2               |
| x          | execute file                         | 1               |
---> If the file doesn't have an `x`, the kernel **will not allow it to run**, regardless of whether the content is a script or a binary.

EX:
```bash
chmod 644 myfile
# Owner: 6 → r+w  
# Group: 4 → r
# Others: 4 → r
```

### 2.1.3 Permission trên DIRECTORY

| permisson | Numerical value | meaning                           |
| --------- | --------------- | --------------------------------- |
| r         | 4               | View the list of file names(`ls`) |
| w         | 2               | create, delete, rename file       |
| x         | 1               | go to directory(`cd`)             |
| -         | 0               | not allowed                       |
--> `x` This is the most important permission on a directory.
EX:
```bash
chmod 755 myfile
# Owner: 7 -> rwx
# Group: 5 -> r-x
# Other: 5 -> r-x
# ---> No execute (`x`) → file doesn't run / directory inaccessible
```

> [!note]
> Key rights combination::
> `--x` → I was able to access it, but I couldn't list anything.
> `r-x` → read directory (read-only)
> `-wx` → Deleting a file without seeing the list is dangerous.
> --> Permission directories control the file path, not its contents.
## 2.2 Ownership Permissions
- Each file always has:
	- Owner(UID)
	- Group(GID)
```bash
ls -l myfile
-rwxr-xr-- 1 alice dev 1234 myfile
# Owner: alice
# Group: dev
```


- Change owner – `chown`

```bash
chown bob myfile              # Change owner
chown bob:dev myfile          # Change owner + group
sudo chown root:root myfile   # Only rooted devices can do this.
# --> Users are usually unable to change the owner.
```

- Change group – `chgrp`
```bash
chgrp dev myfile
# Users can only switch to the group they belong to.
```

## 2.3 UID trong process – Who decides the right??

- Each process has:

| UID       | role                                         |
| --------- | -------------------------------------------- |
| EUID      | Kernels are used for privilege verification. |
| RUID      | Who is running the process?                  |
| Saved UID | Backup ticket to regain setuid privileges    |
--> <font color="#ffff00">**Permission check = EUID**</font>
## 2.4 setuid
- <font color="#ffff00">**This is the permission bit on the file.**</font>
- When the file runs, the kernel:
    - keep RUID
    - **Change EUID = owner UID of the file**

```bash
-rwsr-xr-x root root myprog

# --> process running with root privileges
```

> [!note]
> The nature of writing`s`:
> - The letter `s` does not grant permission to the USER, but rather allows the process to temporarily borrow the file owner's permissions.
> - Kernel trusts the process, not the user.
> - Power is temporary, intended for specific actions.

## 2.5 Saved UID
- Known UID is used for:
	- for process <font color="#ffff00">**drop privilege**</font>
	- the <font color="#ffff00">**regain privilege**</font> when needed.
-  <font color="#ffff00">setuid Cycle:</font>
```bash
RUID   = user
EUID   = root
Saved  = root
```

- `seteuid(user)` →lower power
- `seteuid(root)` → return to rights
--> **The power is not lost, it's just dormant.**

