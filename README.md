# ad-security-homelab
# Active Directory Security Home Lab

A self-contained Active Directory environment built on KVM/libvirt, used to practice
AD administration, systematic troubleshooting, and offensive security fundamentals.

## Why this project

Built to demonstrate practical skills relevant to IT support and security roles:
Active Directory administration, Group Policy management, network troubleshooting,
and understanding how attacks work from both the defender's and attacker's side.

## Architecture

- **Host:** Fedora Linux (KVM/libvirt), isolated virtual network (`lab-net`, 192.168.50.0/24)
- **DC01:** Windows Server 2025 — domain controller, DNS, Active Directory Domain Services
- **CLIENT01:** Windows 11 Enterprise — domain-joined workstation
- **Kali01:** Attacker VM — used for enumeration and Kerberoasting

## What's documented

1. [Lab Setup](docs/01-lab-setup.md) — network design, VM builds, domain creation
2. [AD Configuration](docs/02-ad-configuration.md) — OUs, users, groups, GPOs, file shares
3. [Troubleshooting Log](docs/03-troubleshooting-log.md) — real issues hit and resolved
4. [Kerberoasting](docs/04-kerberoasting.md) — full attack walkthrough against a vulnerable service account
5. [Lessons Learned](docs/05-lessons-learned.md)

## Skills demonstrated

- Active Directory Domain Services deployment, OU/GPO design
- NTFS and share permission layering, security group management
- Network troubleshooting: DNS misconfiguration, NIC/firewall profile issues, SMB service outages
- Kerberos internals: clock synchronization requirements, encryption type negotiation, ticket structure
- Offensive security tooling: nmap, enum4linux-ng, netexec, Impacket, hashcat
