

# 06 - Secured Remote Administration - Azure Bastion

## 6.1 Overview

In the previous labs, the Azure Hub-Spoke network, Azure Firewall, Site-to-Site VPN and Point-to-Site VPN were configured to provide secure network connectivity between Azure resources, on-prem  offices and remote users.

Although administrators can now reach the Azure virtual network through a VPN connection, they still require a secure method to remotely manage individual VMs.

Traditionally, administrators connect directly to a VM using SSH (TCP 22) for Linux or Remote Desktop (TCP 3389) for Windows. This requires each VM to have a public IP address and ports exposed to the Internet.

Although we can restrict access, exposing ports is generally avoided in enterprise cloud environments because it increases the attack risks.

### - What is Azure Bastion

Azure Bastion is a **jump host service** for remote administration of Azure cloud VMs.

Instead of exposing management ports, administrators connect to the Bastion service, which then establishes the **RDP** or **SSH** session to the target VM over its private IP address.

Only the Bastion service has a public address. The VM remain completely private.

In this lab, **Azure Bastion Standard** is deployed to provide secure remote administration for both Windows and Linux VMs. The authentication method is also upgraded from local credentials and SSH keys to Microsoft **Entra ID authentication with Azure RBAC**.

---

## 6.2 Objectives

This chapter demonstrates how to:

- Deploy Azure Bastion Standard
- Configure Azure Bastion Native Client
- Upgrade VM authentication to Microsoft Entra ID
- Configure Azure RBAC VM login roles
- Configure Windows Entra ID login
- Configure Linux Entra ID login
- Connect to Windows and Linux VMs through Azure Bastion
- Validate secure administration without public IP addresses

## 6.3 Architecture
Azure Bastion will be deployed in the **Hub VNet** and serves as the central admin entry point for the entire Hub-Spoke network.

Although the Bastion resource resides only in the Hub VNet, it can serves VMs located in both the Hub and Spoke VNet through VNet peerings. As a result, only **a single Bastion host in the hub VNet** is needed. 

In this lab, a single Azure Bastion deployment provides secure access from remote computers to the following VMs:

- Hub Windows VM
- HR VNet Linux VM
- Finance VNet  Linux VM

No VPN connection or public IP of the VM is needed.

> <img title="" src="../images/arch92.jpg" alt="" width="80%">

## 6.4 Deploy Azure Bastion

1) ### Create a Bastion Subnet in the Hub VNet

   The Bastion Host need to be deployed in a dedicated Bastion subnet in the VNet where it resides.

   So we create a Bastion Subnet first

   ```
   vnet-hub → Subnets → + Subnet
   ```

   Subnet IP range: **10.0.255.0/26**

   Subnet Name: **AzureBastionSubnet** (Can't be changed)

   > <img title="" src="../screenshots/B01.jpg" alt="" width="80%">

2) ### Create a Bastion Host in the subnet

    Name: bastion-hub  
    Tier: **Standard**  
    Virtual network: **vnet-hub**  
    Subnet: **AzureBastionSubnet ** 
    Public IP: **Create new** -> Public IP name: **pip-bastion-hub**  
    
	><img title="" src="../screenshots/B02.jpg" alt="" width="80%">

	><img title="" src="../screenshots/B03.jpg" alt="" width="80%">

---

## 6.5 Configure Native Client

Azure Bastion supports both browser-based sessions and Native Client connections. 

Browser-based access launches an RDP or SSH session directly from the Azure portal.

Native Client allows administrators to use their preferred local applications such as:

- Windows Remote Desktop
- OpenSSH

Unlike Point-to-Site VPN, Azure Bastion does not require a dedicated client application. Instead, Native Client uses the  local RDP like Windows Remote Desktop or SSH client like OpenSSH together with Azure CLI to establish a secure connection through the Azure Bastion service.

In this lab, Azure CLI is used to initiate the connection, while Microsoft Remote Desktop and OpenSSH are used to access the target Windows and Linux VMs.

### Prerequisites

Before using Azure Bastion Native Client, the following requirements must be met.

For Windows VM

\- Remote Desktop (RDP) must be enabled.
\- Windows Firewall must allow RDP connections.

For Linux VM:

\- SSH service must be running.
\- SSH access must be permitted by the operating system firewall

In addition, the Network Security Group (NSG) associated with the virtual machine or subnet must allow the required management traffic from Azure Bastion.



1. ### Enable Native Client Support in Bastion Host

​	In order to use Native Client to make connection to Bastion is to enable the Native Client Support of Bastion Host. 

**Native Client Support**: Enabled

> <img title="" src="../screenshots/B04.jpg" alt="" width="60%">

---

# Upgrade Virtual Machine Authentication

Earlier chapters intentionally used the default authentication methods created during VM deployment.

Linux virtual machines were accessed using SSH key authentication.

The Windows virtual machine used a local administrator account.

This simplified the initial deployment while validating the network infrastructure.

After Azure Bastion is operational, authentication is upgraded to Microsoft Entra ID.

Benefits include:

- Centralized identity management
- Multi-Factor Authentication (MFA)
- Azure RBAC integration
- No shared administrator passwords
- Enterprise identity governance

---

# Configure Windows Entra ID Sign-in

Requirements

- Enable Login with Microsoft Entra ID
- Enable System Assigned Managed Identity
- Assign Virtual Machine Administrator Login role

(Add screenshots)

Explain how Windows authentication changes from local accounts to Entra ID.

---

# Configure Linux Entra ID Sign-in

Linux requires several additional components.

| Area             | Requirement                                      |
| ---------------- | ------------------------------------------------ |
| VM Identity      | System Assigned Managed Identity                 |
| VM Extension     | AADSSHLoginForLinux                              |
| Extension Status | Succeeded                                        |
| RBAC             | Virtual Machine Administrator Login / User Login |

(Add screenshots)

Explain why Linux requires the Azure AD SSH Login extension.

---

# Configure Azure RBAC

Unlike Azure resource permissions, operating system sign-in requires dedicated RBAC roles.

Owner or Contributor permissions alone do not grant login access.

Assign either:

- Virtual Machine Administrator Login
- Virtual Machine User Login

(Add screenshots)

---

# Connect Using Azure Bastion Native Client

## Windows

Demonstrate:

- Azure CLI
- RDP
- --enable-mfa

(Add screenshots)

---

## Linux

Demonstrate:

- Azure CLI
- SSH
- --auth-type AAD

(Add screenshots)

---

# Validation

Verify:

- Windows VM has no public IP
- Linux VMs have no public IP
- TCP 22 not exposed
- TCP 3389 not exposed
- Windows Entra ID login succeeds
- Linux Entra ID login succeeds
- Azure RBAC controls OS login permissions

(Add screenshots)

---

# Summary

Azure Bastion provides secure administrative access without exposing virtual machines directly to the Internet.

Combined with Microsoft Entra ID authentication and Azure RBAC, the solution delivers centralized identity management, strong authentication and enterprise-grade remote administration while maintaining a private Hub-Spoke architecture.



============

5. **Configure Entra ID sign-in for VMs**
   
   As metioned above, both Linux and Windows VMs were initially deployed using traditional authentication methods. The Linux VMs used SSH key authentication, while the Windows VM used a local administrator account and password.
   
   To improve security , user experience and simplify identity management, we change the authentication method to Entra ID

#### Linux VM Entra ID Sign-in Requirements

```
To use Entra ID authentication for Linux VM sign-in, the following requirements must be met.
```

| Area             | Requirement                                                                                                                                 |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| VM Identity      | **System assigned managed identity** must be enabled on the Linux VM.                                                                       |
| VM Extension     | The `AADSSHLoginForLinux` extension must be installed.                                                                                      |
| Extension Status | The extension status must show **succeeded**. If the status is `Installing`, `Transitioning` or `Failed`, Entra ID SSH login will not work. |
| RBAC Role        | The Entra user must be assigned role of either `Virtual Machine Administrator Login` or `Virtual Machine User Login`.                       |

- Install Login Extension for Linux VM
  
  vm-hr-linux1→settings→ Extensions + applications→ Add→ Azure AD Based SSH Login

![](../screenshots/8linux.jpg)

Owner or Contributor permission does not automatically grant operating system login access to the Linux VM. The VM login role must be assigned separately.

In this lab, `Virtual Machine Administrator Login` is assigned to the lab administrator account for testing.
