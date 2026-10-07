# Cybersecurity Learning - Day 2

## Topic
Linux Security & System Enumeration

## Users and Groups

User:
user_a

The user belongs to several groups including:
- sudo
- adm
- lxd
- libvirt
- lpadmin

`sudo -l` showed:

(ALL : ALL) ALL

This means the user can use sudo to run commands with administrative privileges.

## Network Enumeration

Listening services were examined with:

sudo ss -lntup

Important observations:
- dnsmasq on the libvirt virtual network
- systemd-resolved on localhost DNS addresses
- chronyd on localhost
- Avahi on port 5353
- containerd on localhost
- CUPS on localhost port 631

Binding addresses are important when determining network exposure.

## SUID and SGID

SUID files were enumerated with:

find / -perm -4000 -type f 2>/dev/null

SGID files were enumerated with:

find / -perm -2000 -type f 2>/dev/null

Several standard Linux SUID programs were found.

A notable additional SUID executable was:

/opt/LM-Studio/chrome-sandbox

Selected SUID files were inspected using `ls -l` and `file`.

Important lesson:

A SUID file is not automatically a vulnerability. It must be investigated for ownership, permissions, purpose, version, configuration, and possible vulnerabilities.

## World-Writable Files

Searches were performed in:

/etc
/usr/local
/opt

No world-writable files or directories were found in those locations.

## Environment Variables

Environment variables were examined with:

env

Important variables examined included:

PATH
HOME
USER
SHELL
PWD

The PATH contained both system directories and user-owned application directories.

The system PATH directories were root-owned and not writable by the normal user.

## PATH Security

PATH entries were displayed individually and their directory permissions were checked.

User-owned PATH directories included:

/home/user_a/.lmstudio/bin
/home/user_a/.local/share/JetBrains/Toolbox/scripts

No obvious writable system PATH directory was found.

## Command Resolution

`python` was not available as a command.

`python3` resolved to:

/usr/bin/python3

`sudo` resolved to:

/usr/bin/sudo

`ls` is configured as a shell alias and resolves to /usr/bin/ls.

## System Security Checks

OS:

Ubuntu 26.04 LTS

Kernel:

7.0.0-22-generic

`systemctl --failed` reported no failed units.

UFW status:

inactive

AppArmor:

The AppArmor kernel module is loaded.

## Day 2 Lessons

1. Enumerate users and groups.
2. Check sudo privileges.
3. Identify running processes.
4. Identify listening network services.
5. Map ports to processes.
6. Enumerate SUID and SGID programs.
7. Look for dangerous writable files/directories.
8. Inspect environment variables and PATH.
9. Identify OS and kernel information.
10. Check firewall and security controls.

Security enumeration is about understanding the system before attempting to identify vulnerabilities.
