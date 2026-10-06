# Cybersecurity Learning - Day 1

## Goal

Learn basic Linux administration and the first steps of security investigation.

## Linux basics

- `pwd` shows the current working directory.
- `whoami` shows the current account username.
- `hostname` shows the computer's hostname.
- `uname -a` displays detailed kernel/system information.
- `/etc/os-release` contains Linux distribution information.
- `id` shows UID, GID, and group membership.
- `ls -la` shows files including hidden files.
- `stat` displays detailed file metadata and permissions.

## Linux permissions

Permissions use:

- Read = 4
- Write = 2
- Execute = 1

For example:

`600` = owner can read/write, group and others have no permissions.

## Users and password information

`/etc/passwd` contains account information such as:

- username
- UID
- GID
- home directory
- login shell

`/etc/shadow` contains password hashes and password-aging information and is much more restricted.

## Processes

A process is a running program.

A PID (Process ID) identifies a running process.

`kill PID` sends a signal to a process. The default signal is SIGTERM; it does not necessarily immediately force the process to terminate.

## Network ports

A port is a communication endpoint.

`ss -tuln` shows listening/network sockets.

`ss -lntup` provides more information including processes when permissions allow.

Port 22 is commonly associated with SSH.

An unexpected listening port should not automatically be assumed to be malware. Investigate:

1. Port
2. PID
3. Process
4. Executable
5. Parent process
6. Service configuration
7. Network exposure

## Day 1 investigation

### CUPS

Nmap found:

`631/tcp open ipp`

Investigation showed:

`PID 2176 -> cupsd -> CUPS`

CUPS was listening only on:

`127.0.0.1:631`
`[::1]:631`

Therefore the listener was bound to loopback rather than all IPv4 interfaces.

### DNS

`systemd-resolved` was found listening on:

`127.0.0.53:53`
`127.0.0.54:53`

PID:

`1133`

Process:

`/usr/lib/systemd/systemd-resolved`

Parent:

`systemd(1)`

Service:

`systemd-resolved.service`

Status:

`active (running)`
`enabled`

`active` means the service is running now.

`enabled` means it is configured to start automatically according to the system's boot/service configuration.

### dnsmasq

Another DNS listener was found at:

`192.168.122.1:53`

Nmap identified:

`dnsmasq 2.92`

This was associated with the virtual bridge interface and was not the same DNS listener as `systemd-resolved`.

## Important investigation workflow

A useful basic SOC/network investigation chain is:

Network socket
→ Port
→ PID
→ Process
→ Executable
→ Parent process
→ Service/system context

## Day 1 lessons

The main lesson is not simply memorizing commands.

The goal is to learn how to investigate an unfamiliar process or network service systematically instead of immediately assuming that it is malicious.
