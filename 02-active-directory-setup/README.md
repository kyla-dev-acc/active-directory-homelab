# 02 - Active Directory Setup

This folder documents how I installed Active Directory Domain Services on **DC01**, created the `abc.local` domain, and connected **CLIENT01** to it. It builds on the virtual network from [01-lab-environment](../01-lab-environment/).

## Lab Summary

| Item | Details |
|------|---------|
| Domain | `abc.local` |
| Domain Controller | DC01, 192.168.10.10 (AD DS + DNS) |
| Client | CLIENT01, 192.168.10.20, DNS: 192.168.10.10 |
| Network | Internal network `ABC-LAN` (192.168.10.0/24) |

## Contents

| File | Description |
|------|-------------|
| [domain-controller-setup.md](domain-controller-setup.md) | Preparing DC01 as the domain controller |
| [ad-ds-installation.md](ad-ds-installation.md) | Installing the AD DS and DNS Server roles |
| [domain-creation.md](domain-creation.md) | Promoting DC01 and creating the `abc.local` domain |
| [client-domain-join.md](client-domain-join.md) | Joining CLIENT01 to the domain |

## What I Accomplished

- Installed the Active Directory Domain Services role and DNS Server on DC01.
- Promoted DC01 to a domain controller for the new `abc.local` domain.
- Joined CLIENT01 to the domain, using DC01 as its DNS server.

## Skills Demonstrated

- Installing and configuring server roles with Server Manager
- Active Directory and DNS fundamentals
- Domain controller promotion and domain creation
- Joining a Windows client to a domain

## Next

Continue to [03-user-and-group-management](../03-user-and-group-management/) to build the OU structure and create user accounts.
