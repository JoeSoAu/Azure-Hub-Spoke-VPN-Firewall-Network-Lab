# Azure Hub-Spoke VPN Firewall Network-Lab

## Overview

This lab demonstrates the design and implementation of a compact modern enterprise Azure Hub-Spoke cloud network using the following Azure networking, security and identity technologies:

- **Hub-Spoke Vnet** desgin for centralized shared services and departmental network isolation.
- **VNet peering** for inter-communication between vnets
- **Azure Firewall** to control Internet-bound and inter-spoke network traffic.
- **User Defined Routes (UDRs)** to direct traffic through VPN tunnels and Firewall.
- **Network Security Groups (NSGs)** for subnet and VM access control.
- **Entra ID** & **RBAC** for centralized authentication and Permissions.
- **Azure Bastion** for secure RDP and SSH access without exposing VM to the Internet.
- **Azure VPN Gateway** supporting both Site-to-Site and Point-to-Site VPN connections.
- **Site-to-Site IPsec VPN** tunneling between an on-prem AD to Azure.
- **Point-to-Site VPN** providing secure remote access for mobile users.
- **Gateway Transit** to provide shared VPN Gateway access for spoke VNets.
- **Windows Server 2022 RRAS** providing the on-premises VPN gateway.

---

## Architecture

![Architecture](/images/arch.jpg)

---

## Documentation

| Document                                                     | Description                                                |
| ------------------------------------------------------------ | ---------------------------------------------------------- |
| [01 - Overview and Design](docs/01-Overview-and-Desgin.md)   | Network design and VNet deployment                         |
| [02 - Hub-Spoke Virtual Networking](docs/02-Hub-Spoke-Virutal-Network.md) | Preparing Hub-Spoke Virtual Networking                     |
| [03 - Centralized Network Security Azure Firewall](docs/03-Centralized-network-security-firewall.md) | Deploy Azure Firewall as central Security Appliance        |
| [04 - Hybrid Connectivity Site-to-site VPN](docs/04-hybrid-connectivity-site-to-site-vpn.md) | Deploy S-S VPN between on-prem and cloud                   |
| [05 - Remote User Connectivity Point-to-site VPN](docs/05-Remote-User-Connectivity-Point-to-Site-VPN.md) | Deploy P-S VPN between remote device and cloud             |
| [06 - Secured Remote Administration Bastion](docs/06-Secured-Remote-Administration-Bastion.md) | Deploy Azure Bastion Service for remote access to cloud VM |
| [07 - Troubleshooting](docs/07-Troubleshooting.md)              | Problems encountered and solutions                         |

---

## Lab Environment

### Azure Cloud

**Hub VNet**

- Vnets

- Azure Firewall

- Azure Bastion

- VPN Gateway

- Windows 10 VM

**Finance Spoke**

- Linux VM

**HR Spoke**

- Linux VM

### On-Premises Network

- Windows Server 2022 Domain Controller

- RRAS VPN Server

- Home Router (for Port Forwarding)

- Windows Workstation

### Remote Users

- Azure VPN Client (Point-to-Site VPN)

- Azure Bastion Native Client

- Microsoft Entra ID Authentication

---

## Features

- A Central Hub Vnet + 2 isolated Finance and HR spoke VNets.  

- VNet Peering with Gateway Transit and One-way traffic control

- 2 Private spoke vnet without public IP addresses and Internet gateway

- Centralized Internet access through Azure Firewall in the hub

- Traffic control between spoke VNets using UDRs and NSGs and Azure Firewall. 
  (Only SSH TCP 22 allowed from Finance spoke to HR, all other traffics restricted)

- Secure administration of Windows and Linux virtual machines without public IP addresses.  

- VM access by Bastion Native Client with Entra ID authentication

- Hybrid connectivity between Azure and an on-premises Active Directory network using Site-to-Site VPN.  

- Secure remote user access using Point-to-Site VPN.  

- Shared VPN Gateway access for spoke VNets using Gateway Transit.

- Using Azure Firewall controlling Internet and inter-spoke traffic

- UsingUDR, NSGs for traffic control

- Using Windows Server 2022 RRAS as the on-prem VPN router

---

## Technologies

- Azure Virtual Network

- Vnet Peering with gateway transit and Forwarded traffic control

- User Defined Routes (UDR)

- Network Security Groups (NSG)

- Azure Firewall

- Azure Bastion

- Azure VPN Gateway

- Site-to-Site VPN

- Point-to-Site VPN

- Entra ID

- Azure RBAC

- Windows Server 2022

- RRAS

- Router Port forwarding ( Virtual Server)

- Windows 10

- Ubuntu Linux

- Hyper-V

- Powershell Cmdlets

---

## Troubleshooting

