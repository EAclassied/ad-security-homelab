# Lab Setup

## Host environment

- Fedora Linux laptop (ASUS Q540V) running KVM/libvirt via `virt-manager`
- Full-disk encryption (LUKS) on the host
- Secure Boot disabled — Kali's pre-built QEMU image was not UEFI/Secure-Boot
  compatible, and this is a common limitation of prebuilt security distro images

## Network design

An isolated virtual network (`lab-net`, 192.168.50.0/24) was created in virt-manager,
with no route to the host's home network. This keeps all AD traffic, intentionally
vulnerable configurations, and attack tooling fully contained.

| Host | IP | Role |
|---|---|---|
| DC01 | 192.168.50.10 | Domain controller (AD DS, DNS) |
| CLIENT01 | 192.168.50.20 | Domain-joined Windows 11 client |
| Kali01 | 192.168.50.30 | Attacker/enumeration machine |

DC01 and CLIENT01 each also have a second NIC on the default NAT network for
internet access (Windows Updates, downloading tools). Kali01 initially had only
the isolated NIC and had the NAT NIC added later when internet access was needed
to install tooling.

## VM builds

All VMs run under KVM/libvirt with UEFI firmware, except Kali01, which required
switching to legacy BIOS firmware (see Troubleshooting Log) since its pre-built
disk image was not Secure-Boot/UEFI compatible.

- **DC01:** Windows Server 2025, 4GB RAM, 2 vCPU, 60GB disk, SATA bus / e1000e NIC
- **CLIENT01:** Windows 11 Enterprise (evaluation), 4GB RAM, 2 vCPU, 60GB disk,
  SATA bus / e1000e NIC, emulated TPM 2.0 (required for Windows 11)
- **Kali01:** Pre-built Kali QEMU image, 4GB RAM, 2 vCPU, VirtIO disk/NIC, legacy BIOS

## Domain creation

DC01 was promoted to a domain controller for a new forest:

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName "lab.internal" -DomainNetbiosName "LAB" -InstallDns
```

Domain: `lab.internal` (NetBIOS: `LAB`). A non-`.local` domain name was chosen
deliberately to avoid conflicts with mDNS.

CLIENT01 was joined to the domain after static IP/DNS configuration pointing at DC01:

```powershell
Add-Computer -DomainName "lab.internal" -Credential LAB\Administrator -Restart
```
