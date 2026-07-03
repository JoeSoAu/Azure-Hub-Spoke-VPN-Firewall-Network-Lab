# 3. Centralized Network Security (Firewall)

## Overview

After establishing the Hub-Spoke network topology, the next step is to centralize network traffic control and security configuration using **Azure Firewall,** **User Defined Routes (UDR)** and **Network Security Groups (NSG)**.

Rather than allowing each spoke VNet to access the Internet independently, this lab routes outbound traffic through a **centralized Azure Firewall** deployed in the Hub VNet. Additionally, **all inter-spoke VNet traffic** is redirected to the firewall for centralized control and inspection before reaching its destination.

In this chapter, Azure Firewall is deployed in the Hub VNet. **User Defined Routes (UDRs)** are then configured to redirect outbound traffic from the spoke VNets to the firewall. Finally, **Firewall Policies** are
created to control Internet access and inter-spoke communication.

><img src="..\images\arch8.jpg" width="90%"/>

------------------------------------------------------------------------

## Objectives

The objectives of this chapter are to:

-   Deploy Azure Firewall in the Hub VNet.
-   Centralize outbound Internet access.
-   Route spoke network traffic through Azure Firewall using User
    Defined Routes (UDRs).
-   Control Internet and inter-spoke communication using Firewall
    Policies.
-   Validate that all traffic is inspected before reaching its
    destination.

------------------------------------------------------------------------

## Deploy Azure Firewall

Azure Firewall is deployed inside the Hub VNet. An Azure Firewall needs a dedicated
**AzureFirewallSubnet**. Here is the parameters for the Firewall subnet and firewall

| Resource           |               |
| ------------------ | ------------- |
| Firewall Subnet IP | 10.0.254.0/26 |
| Firewall IP        | 10.10.254.4   |
| Firewall SKU       | Standard      |

#### 1 Create a Firewall Subnet in Hub VNet
The first step is to prepare a firewall subnet in Hub Vnet

IP address: 10.0.254.0/16

> <img src="..\screenshots\31firewallnet.jpg" width="90%" />

#### 2 Create an Azure Firewall (SKU Standard)
Then create an **Azure firewall** with the parameters shown in the screenshot

> <img src="..\screenshots\32firewall.jpg" width="50%" />

A **public IP** is created for the firewall: ***pip-firewall-hub***

><img src="..\screenshots\33ip.jpg" width="70%"/>






------------------------------------------------------------------------

## Configure User Defined Routes

By default, Azure routes Internet-bound traffic directly to the
Internet. To force traffic through Azure Firewall, User Defined Routes
(UDRs) are created for each spoke subnet.

Each route table contains a default route (`0.0.0.0/0`) with the next
hop configured as **Virtual Appliance**, pointing to the private IP
address of Azure Firewall.

The route tables are then associated with the Finance and HR application
subnets.

Once applied, all outbound traffic from the spoke subnets is redirected
to Azure Firewall before leaving the virtual network.

> **Insert:** Route Table and Route screenshots

------------------------------------------------------------------------

## Configure Firewall Policy

After traffic is redirected to Azure Firewall, Firewall Policies
determine whether the traffic should be permitted or denied.

This lab uses Network Rule Collections to allow required outbound
Internet access and selected inter-spoke communication. Future security
policies can be managed centrally without modifying individual spoke
VNets.

> **Insert:** Firewall Policy and Rule Collection screenshots

------------------------------------------------------------------------

## Validation

After the firewall configuration is completed, verify the following:

-   Internet access from both spoke virtual machines.
-   Traffic is routed through Azure Firewall.
-   Inter-spoke communication follows the configured Firewall Policy.
-   Traffic without a matching rule is blocked.

Successful validation confirms that centralized routing and security
inspection are functioning as designed.

------------------------------------------------------------------------

## Summary

Azure Firewall has been deployed as the central security appliance for
the Hub-Spoke network. User Defined Routes redirect traffic from the
spoke VNets to the firewall, while Firewall Policies determine whether
traffic is permitted to continue to its destination.

This centralized security architecture provides the foundation for the
hybrid connectivity and secure remote administration features
implemented in the following chapters.
