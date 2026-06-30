# 01 - Overview and Network Design

## Architecture Design

This lab builds a compact Azure hybrid cloud network that simulates a typical enterprise Azure cloud + on-premises virtual network.

The Network uses a Hub-Spoke architecture. Shared infrastructure and services are centralized in a Hub VNet, while departmental workloads are deployed in isolated spoke VNets. The lab consists of one Hub VNet and two Spoke VNets representing Finance and HR departments.

Instead of allowing unrestricte traffic between all virtual networks, each department is logically isolated. Shared services are accessed through the Hub, and any traffic between spokes must be explicitly permitted through centralized firewall rules.

The lab also demonstrates hybrid connectivity by integrating the on-prem network with Azure cloud VNets using Site-to-Site VPN, while roaming users can securely access Azure resources through Point-to-Site VPN. Remote administration is implitented using Azure Bastion technology without exposing VMs directly to the Internet.

The objective of this lab is to demonstrate how enterprise Azure networking can provide centralized management, network isolation, secure remote access and controlled hybrid connectivity using a compact and practical deployment.

![Architecture](/images/arch2.jpg)

---

## 

## High-Level Architecture

The Azure cloud network is made up of three VNets:

| VNet               | Role                       | Main Resources                                            |
| ------------------ | -------------------------- | --------------------------------------------------------- |
| Hub VNet           | Shared network services    | Azure Firewall, Azure Bastion, VPN Gateway, Windows 10 VM |
| Finance Spoke VNet | Finance department network | Finance Linux VM                                          |
| HR Spoke VNet      | HR department network      | HR Linux VM                                               |

The Hub VNet is peered with both spoke VNets.

There is no direct peering between Finance and HR.

---

## Azure VNet Layout

### Hub VNet

The Hub VNet provides the shared services used by the whole Azure network.

Main components in the Hub:

- Azure Firewall

- Azure Bastion

- Azure VPN Gateway

- Windows 10 management VM

Example Hub subnets:

| Subnet              | Purpose                  |
| ------------------- | ------------------------ |
| Management subnet   | Windows 10 management VM |
| AzureFirewallSubnet | Azure Firewall           |
| AzureBastionSubnet  | Azure Bastion            |
| GatewaySubnet       | Azure VPN Gateway        |

The Hub is used as the central point for:

- Internet traffic control

- Inter-spoke traffic control

- Site-to-Site VPN

- Point-to-Site VPN

- Bastion access to private VMs

---

### Finance Spoke VNet

The Finance Spoke VNet represents a department network.

It contains:

- Finance Linux VM

The Finance VNet is peered with the Hub VNet.

Finance does not have direct peering with HR.

---

### HR Spoke VNet

The HR Spoke VNet represents another department network.

It contains:

- HR Linux VM

The HR VNet is peered with the Hub VNet.

HR does not have direct peering with Finance.

---

## VNet Peering Design

The peering design is:

| Peering       | Purpose                                   |
| ------------- | ----------------------------------------- |
| Hub ↔ Finance | Allows Finance to use shared Hub services |
| Hub ↔ HR      | Allows HR to use shared Hub services      |
| Finance ↔ HR  | Not configured                            |

Because Finance and HR are not directly peered, they cannot communicate directly.

If Finance needs to access HR, traffic must be routed through the Hub and controlled by Azure Firewall and NSG rules.

---

## Spoke-to-Spoke Traffic Design

Finance and HR are separated because they represent different departments with different access requirements.

For example:

- HR systems should not have unrestricted access to Finance systems.

- Finance systems should not have unrestricted access to HR systems.

In this lab:

| Traffic                    | Result  |
| -------------------------- | ------- |
| Finance → HR SSH TCP 22    | Allowed |
| Finance → HR other traffic | Blocked |
| HR → Finance traffic       | Blocked |

The only permitted inter-spoke communication is:

```text
Finance Linux VM → HR Linux VM
Protocol: SSH
Port: TCP 22
```

All other traffic between Finance and HR is blocked by Azure Firewall rules and NSGs.

---

## Internet Traffic Design

Azure VMs do not access the Internet directly.

Internet-bound traffic from Hub, Finance and HR is routed through Azure Firewall using User Defined Routes.

The intended traffic path is:

```text
Azure VM
  → Subnet Route Table
  → Azure Firewall
  → Internet
```

This allows outbound traffic to be controlled by firewall rules.

---

## On-Premises Connectivity

The on-premises LAN is connected to Azure using a Site-to-Site IPsec VPN.

On-premises components:

| Component                             | Purpose                                 |
| ------------------------------------- | --------------------------------------- |
| Windows Server 2022 Domain Controller | On-prem AD DS server                    |
| RRAS                                  | Site-to-Site VPN router                 |
| Home Router                           | NAT and port forwarding for VPN traffic |
| Workstation PC                        | On-prem client used for testing         |

The Site-to-Site VPN connects the on-premises network to the Azure VPN Gateway in the Hub VNet.

Traffic path:

```text
On-prem workstation
  → On-prem router
  → Windows Server 2022 RRAS
  → Site-to-Site VPN tunnel
  → Azure VPN Gateway
  → Azure VNets
```

The on-premises workstation was used to test access to Azure Hub and Spoke VMs.

---

## Point-to-Site VPN Design

Point-to-Site VPN is used for remote users outside the on-premises network.

A remote laptop connects to Azure using Azure VPN Client.

Traffic path:

```text
Remote laptop
  → Azure VPN Client
  → Point-to-Site VPN
  → Azure VPN Gateway
  → Azure Hub / Spoke VNets
```

After connecting, the remote laptop can access private Azure VM IP addresses without exposing the VMs to the Internet.

---

## Bastion Access Design

Azure virtual machines do not have public IP addresses.

Administrative access is provided through Azure Bastion Native Client.

Supported access paths:

| Target VM         | Access Method             |
| ----------------- | ------------------------- |
| Hub Windows 10 VM | RDP through Azure Bastion |
| Finance Linux VM  | SSH through Azure Bastion |
| HR Linux VM       | SSH through Azure Bastion |

Microsoft Entra ID authentication is used for Bastion-based VM access.

This allows administrators to manage cloud VMs without opening public RDP or SSH ports.

---

## Routing and Security Control

The lab uses multiple layers of routing and security control.

| Component         | Role                                                |
| ----------------- | --------------------------------------------------- |
| UDR               | Sends traffic through Azure Firewall or VPN Gateway |
| Azure Firewall    | Controls Internet and inter-spoke traffic           |
| NSG               | Provides subnet and VM-level access control         |
| VNet Peering      | Connects Spokes to Hub                              |
| Gateway Transit   | Allows Spokes to use the Hub VPN Gateway            |
| Azure Bastion     | Provides private VM administration                  |
| Entra ID and RBAC | Controls who can log in and administer VMs          |

---

## Final Design Summary

The final design provides:

- One Hub VNet for shared services

- Two isolated Spoke VNets for Finance and HR

- No direct Finance-to-HR peering

- Controlled Finance-to-HR SSH access only

- Azure Firewall for Internet and inter-spoke traffic control

- No public IP addresses on Azure VMs

- Azure Bastion Native Client for RDP and SSH

- Site-to-Site VPN from on-premises to Azure

- Point-to-Site VPN for remote users

- Shared VPN Gateway access through Gateway Transit
