# Day 04 — Network Exposure and Firewall Review

## Objective

Identify locally listening network services, document the host firewall posture, and distinguish required services from unnecessary exposure.

## Scope

This is a defensive review of a personally administered Linux workstation. All checks are local and read-only. No port scans, firewall changes, service restarts, or configuration changes are performed.

## Evidence collected

### Network baseline

- The active connection used a private IPv4 address on the wireless interface and a private-network default gateway.
- The wired interface was down.
- Docker and Libvirt virtual-network bridges were present but reported as down.

### Listening-service review

- No TCP listener was bound to the active wireless interface or to all IPv4/IPv6 interfaces.
- Local DNS resolver listeners were bound only to loopback addresses.
- Printing and development-tool listeners were bound only to loopback addresses.
- DNS and DHCP listeners were bound to the inactive Libvirt virtual-network bridge, which is expected for local virtual-machine networking.
- Multicast DNS (UDP 5353) was active for local-network service discovery. This is normal on many desktop Linux installations, but it is the only observed listener that is discoverable on the local network.
- The privileged socket listing identified expected local services: `systemd-resolved` for loopback DNS, `chronyd` for local time synchronization, `cupsd` for loopback printing, `containerd` for a loopback-only container endpoint, and local development-tool listeners.
- `dnsmasq` provided DNS and DHCP only on the Libvirt virtual-network bridge.
- `avahi-daemon` owned the multicast-DNS listeners on UDP 5353.
- The desktop application process also owned several UDP sockets, including wildcard-bound sockets. Their purpose should be confirmed if local-network discovery is not required.

### Firewall state

- UFW reported `Status: inactive`; no UFW policy is currently enforcing host-level filtering.
- nftables input chains for both IPv4 and IPv6 had an `accept` default policy. The ruleset therefore does not provide a default-deny inbound host firewall.
- Docker and Libvirt rules were present for container/virtual-machine forwarding and source-NAT. The captured Docker rules showed no published-port rules, and their forwarding counters were zero at the time of review.
- Libvirt rules allow DNS/DHCP on its virtual bridge and provide outbound NAT for the virtual network. These rules support local virtualization; they are not a general host-firewall policy.

## Preliminary assessment

**TCP exposure: Pass.** No TCP listener was bound to the active wireless interface or to all interfaces.

**Firewall posture: Improvement recommended.** UFW is inactive and nftables has a default-accept input policy for IPv4 and IPv6. Multicast-DNS and application UDP listeners are visible beyond loopback. No published Docker port was observed, but a default-deny inbound policy would provide a stronger baseline.

### Service-intent verification

- SSH was not found as a systemd unit and was inactive. No SSH remote-access service was enabled.
- `avahi-daemon` was enabled and active. This explains the observed multicast-DNS traffic and represents an intentional local-network discovery service.

## Recommendations

1. Define which local-network services are required before enabling any firewall policy.
2. Use a default-deny inbound policy with explicit allow rules only for needed services; verify the policy locally before depending on it for remote administration.
3. Keep mDNS enabled only when local-network service discovery is required; otherwise, disable the Avahi service.
4. Recheck listening sockets after starting containers, virtual machines, or new desktop applications.
5. Document any intentional port exposure in this report.

## Assessment method

1. Record active network interfaces and the default route.
2. List listening TCP and UDP sockets.
3. Compare unprivileged and privileged socket listings to identify service ownership.
4. Inspect UFW and nftables rules.
5. Confirm whether remote-access services are running and intentionally enabled.

## Success criteria

- Every listening service has an identified purpose.
- Unneeded remote-access services are disabled or restricted.
- Firewall policy is documented and appropriate for the host's use.
- Findings and recommendations are based on collected evidence.
