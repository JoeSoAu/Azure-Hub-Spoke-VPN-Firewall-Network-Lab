

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

> <img title="" src="../images/arch92.jpg" alt="" width="80%">



## 6.3 Deploy Azure Bastion

Describe:

- AzureBastionSubnet
- Standard SKU
- Public IP attached only to Bastion
- Native Client enabled



---

# Native Client

Azure Bastion supports both browser-based access and Native Client connections.

Browser-based access launches an RDP or SSH session directly from the Azure portal.

Native Client allows administrators to use their preferred local applications such as:

- Windows Remote Desktop
- OpenSSH
- Visual Studio Code Remote SSH

This lab uses Native Client because it provides a more natural administration experience while still maintaining Bastion security.

(Add screenshots)

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
