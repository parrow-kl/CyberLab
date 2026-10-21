# Day 03 — Linux Privilege Review

## Objective

Perform a defensive, authorized review of local privilege boundaries, scheduled tasks, service configuration, and container/virtualization management interfaces.

## Scope

The checks were limited to a personally administered Linux workstation. Commands were used to inspect configuration and list resources only. No exploit attempts, permission changes, service changes, container changes, or virtual-machine changes were performed.

## Key findings

### Identity and administrative authorization

The assessed account was a standard non-root user but was authorized to use unrestricted `sudo`. This is a deliberate administrative path, not evidence of a vulnerability. It means local privilege-escalation techniques are unnecessary to perform approved administrator tasks.

### Account-file protections

`/etc/passwd` was readable as expected, while `/etc/shadow` was restricted to privileged users and the `shadow` group. This reflects the standard Linux account-password protection model: public account metadata is separated from password hashes.

### File capabilities

The capability scan identified expected system programs, including network utilities, Snap confinement components, and service helpers. A capability is a narrowly scoped privilege and is not automatically a security flaw. Each capability-bearing binary should be considered in the context of its package, configuration, ownership, and patch level.

### Writable-path review

User-owned files and directories under the home directory were writable, which is expected. A writable path becomes security-relevant only if a higher-privileged process trusts, executes, or loads content from it. No obvious privileged dependency on a user-writable path was identified during the reviewed checks.

### Cron and service review

The reviewed cron directories and standard scheduled maintenance entries were owned by privileged accounts and were not writable by ordinary users. The running-service review identified typical desktop and infrastructure services. Their presence alone is not a vulnerability; secure assessment requires confirming that executable paths, configuration files, scripts, and working directories are not writable by lower-privileged users.

### Docker socket control

The Docker Unix socket was owned by `root:docker` and restricted to the owner and group. The normal user session was not a member of the `docker` group, and `docker ps` was denied. The same listing command worked through authorized `sudo`, confirming that Docker administration requires an approved privileged path.

### LXD and Libvirt access

The normal user was a member of the `lxd` and `libvirt` groups and could list local LXD and Libvirt virtual-machine resources without `sudo`. These interfaces are administrative in nature and should be assigned according to least-privilege requirements. Their existence is not, by itself, a vulnerability.

## Assessment

**Docker socket boundary: Pass.** Direct Docker socket access was blocked for the normal user session.

**Privileged-group exposure: Review required.** Active membership in `sudo`, `lxd`, and `libvirt` should be justified by operational need and reviewed periodically.

## Recommendations

1. Keep routine user accounts out of the `docker` group unless direct Docker administration is required.
2. Review `sudo`, `lxd`, `libvirt`, and `docker` memberships on a regular schedule.
3. Remove privileged access when the related administrative responsibility ends.
4. Keep Docker, LXD, Libvirt, Snap, and the operating system current with security patches.
5. Re-run the Day 03 checks after account, service, or group-membership changes.

## Skills demonstrated

- Linux user, group, and sudo-policy enumeration
- Account-file permission validation
- Linux capability review
- Writable-path, cron, and service-permission assessment
- Unix socket access-control testing
- Docker, LXD, and Libvirt privilege-boundary validation
- Evidence-based security reporting
