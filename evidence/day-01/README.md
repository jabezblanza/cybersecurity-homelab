# Day 01 — Infrastructure & Network Configuration

**Project:** Cybersecurity Home Lab — Offensive Security & Incident Response  
**Date:** November 12, 2026  
**Status:** Completed  
**Environment:** Oracle VirtualBox

## 1. Objective

Build and configure a virtualized cybersecurity home lab to support controlled penetration testing, security monitoring, log analysis, and incident response exercises.

The lab uses three virtual machines interconnected through a private Host-only network.

## 2. Lab Environment

| Machine | Operating System | Role |
|---|---|---|
| Kali Linux | Kali Linux | Attacker / Red Team |
| Ubuntu Server | Ubuntu 24.04 LTS | Target / Monitored Endpoint |
| Wazuh Server | Amazon Linux 2023 | SIEM / Blue Team |
| Host | Windows | Virtualization and Management |

**Hypervisor:** Oracle VirtualBox

## 3. Network Architecture

Two virtual network adapters were configured on each VM:

- **NAT:** Provides internet access for software installation and system updates.
- **Host-only:** Provides private communication between the virtual machines and Windows host.

### IP Addressing

| Machine | Host-only IP | Interface |
|---|---|---|
| Windows Host | 192.168.56.8 | VirtualBox Host-only |
| Kali Linux | 192.168.56.10/24 | eth1 |
| Ubuntu Server | 192.168.56.20/24 | enp0s8 |
| Wazuh Server | 192.168.56.30/24 | eth0 |

**Subnet:** 192.168.56.0/24

Each VM also has a NAT connection, used for internet connectivity. NAT will be disabled during isolated attack simulations.

## 4. Implementation

### Ubuntu Server

Configured the Host-only interface with a persistent static IPv4 address using Netplan.

Configuration:

```yaml
network:
  version: 2
  ethernets:
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.56.20/24
```

Validated the configuration using `netplan generate` and `netplan try`.

### Kali Linux

Created a NetworkManager connection profile for the Host-only interface.

```bash
sudo nmcli connection add type ethernet ifname eth1 con-name homelab ipv4.method manual ipv4.addresses 192.168.56.10/24 ipv6.method disabled connection.autoconnect yes
sudo nmcli connection up homelab
```

Verified that the assigned IP address persisted after reboot.

### Wazuh Server

Configured network interfaces using `systemd-networkd`.

Host-only configuration:

```ini
[Match]
Name=eth0

[Network]
DHCP=no
Address=192.168.56.30/24
```

NAT configuration:

```ini
[Match]
Name=eth1

[Network]
DHCP=ipv4
```

Applied the network configuration using `networkctl reload` and `networkctl reconfigure`.

## 5. Verification & Results

| Verification | Result |
|---|---|
| Windows-to-Ubuntu connectivity | Passed |
| Kali-to-Ubuntu connectivity | Passed |
| Ubuntu-to-Kali connectivity | Passed |
| Wazuh-to-Ubuntu connectivity | Passed |
| Wazuh-to-Kali connectivity | Passed |
| Internet and DNS resolution | Passed |
| Static IP persistence after reboot | Passed |
| Wazuh dashboard login page | Accessible |

Intermittent ICMP packet loss was observed during internet tests; this did not prevent successful connectivity.





### Network Configuration Evidence

**Ubuntu Server**

![Ubuntu static IP configuration](../evidence/day-01/ubuntu-network.png)

**Kali Linux**

![Kali static IP configuration](../evidence/day-01/kali-network.png)

**Wazuh Server**

![Wazuh static IP configuration](../evidence/day-01/wazuh-network.png)

### Connectivity Evidence

![Kali to Ubuntu ping test](../evidence/day-01/connectivity-check.png)

### SIEM Dashboard

![Wazuh dashboard](../evidence/day-01/wazuh-dashboard.png)




## 6. Troubleshooting

**Issue 1 — Ubuntu Host-only interface had no IPv4 address.**

Resolution: Configured a static IP through Netplan.

**Issue 2 — Wazuh interface remained unmanaged.**

Cause: Incorrect interface match directive (`Type=eth0`).

Resolution: Corrected the directive to `Name=eth0`.

**Issue 3 — Wazuh NAT interface lost IPv4 configuration.**

Resolution: Created a dedicated DHCP configuration for `eth1`.

**Issue 4 — Wazuh dashboard initially showed "server is not ready yet".**

Resolution: Verified Wazuh services and subsequently confirmed dashboard availability.

## 7. Outcome

Successfully deployed and configured a three-machine virtualized cybersecurity environment.

The environment now supports communication among the attacker, target, and SIEM machines. Static IPv4 configurations persist across reboots, and the Wazuh dashboard is accessible from the Windows host.

## 8. Next Steps

- Install the Wazuh agent on Ubuntu Server.
- Configure endpoint log collection.
- Verify security event ingestion.
- Establish baseline security monitoring.
- Begin controlled attack simulations and incident response exercises.
