# Overview and Network Design

## Architecture Overview (Hub-Spoke Hybrid Cloud)

This lab builds a compact Azure hybrid cloud network that simulates a typical enterprise **Azure cloud + on-premises** hybrid virtual network.

The Network uses a **Hub-Spoke architecture**. Shared infrastructure and services are centralized in a Hub VNet, while departmental workloads are deployed in isolated spoke VNets. The lab consists of one Hub VNet and two Spoke VNets representing Finance and HR departments.

Instead of allowing unrestricte traffic between all virtual networks, each department is logically isolated. Shared services are accessed through the **central Hub**, and any traffic between spokes must be explicitly permitted through **centralized firewall** rules.

The lab also demonstrates hybrid network connectivity by integrating the on-prem network with Azure cloud VNets using **Site-to-Site VPN**, while roaming users can securely access Azure resources through **Point-to-Site VPN**. Remote administration is implitented using **Azure Bastion** technology without exposing VMs directly to the Internet.

The objective of this lab is to demonstrate how enterprise Azure networking can provide centralized management, n

<img title="" src="images/arch2.jpg" alt="" width="">

## Hub-Spoke Network Design (VNet Peering)

The Azure cloud network is built using a Hub-Spoke topology, a common architecture for enterprise environments. Instead of connecting every Vnet directly to one another, shared infrastructure and services are centralized in a dedicated Hub VNet, while application VMs are deployed in separate spoke VNets.

This lab contains one Hub VNet and two spoke VNets representing different business departments: **Finance** and **HR**. Each spoke has its own address space and subnets and is connected only to the Hub through VNet peering. There is **no direct peering** between the two spoke networks.

This design provides logical network isolation between departments while allowing shared services delivered from the central hub. Resources such as Azure Firewall, Bastion and the VPN Gateway are deployed only once in the Hub instead of being duplicated in each spoke, reducing management complexity and deployment cost.

By default, vms in one spoke cannot communicate directly with another spoke because VNet peering is not transitive. When inter-department communication is required, traffic is routed through the Hub and can be explicitly permitted by centralized firewall rules. This approach gives administrators full control of which services are allowed to go through while keeping network seperation between departments. 

<img title="" src="file:///images/arch2" alt="" width="">

## Centralized Network Security (Azure Firewall)

To simplify security management, network security is centralized within the Hub VNet rather than configured independently in each spoke VNet. This approach allows security policies to be managed from a single location and keep consistent protection across the entire Azure network.

**Azure Firewall** is deployed in the Hub as the central security service. User workload traffic that requires inspection is routed through the firewall using **User Defined Routes (UDR).** This includes **Internet-bound traffic** from the spoke networks as well as **selected inter-spoke communication**. 

In this lab, the spoke VNets do not have direct Internet access. Outbound traffic is routed to the Hub and inspected by Azure Firewall before being allowed to reach the Internet. Likewise, communication between the HR and Finance spokes is not permitted by default. Any required connectivity between departments must be explicitly authorized through the Hub.

For simplicity, Site-to-Site and Point-to-Site VPN traffic bypasses Azure Firewall in this lab. Routing VPN traffic through the firewall can be achieved in production environments but requires additional complex routing configuration to ensure **return traffic** follows the same inspection path and avoids **asymmetric routing**.

<img title="" src="file:///images/arch3" alt="" width="">

## Hybrid Connectivity （Site to Site VPN)

To simulate a real enterprise hybrid environment, the on-premises network is connected to Azure cloud Vnets using a **Site-to-Site IPsec VPN**. Rather than putting a VPN gateway in each spoke network, a single Azure VPN Gateway is deployed in the Hub VNet. The spoke VNets use **Gateway Transit** through **VNet peering** to share the Hub gateway, providing secure connectivity to the on-premises network while reducing cost and simplifying network management.

On the on-premises side, Windows Server 2022 RRAS is used as the VPN endpoint. Although production environments typically use dedicated VPN router, RRAS provides a practical software-based solution for demonstrating enterprise VPN technologies in my lab.

Due to the Azure trial subscription being limited to a single public IP address for the VPN Gateway, this lab implements an **active-standby VPN** connection. In production deployments, an **active-active VPN** with two public IP addresses is commonly used to improve availability and better resilience against gateway failures.

<img title="" src="file:///images/arch4" alt="" width="">

## Remote User Connectivity (Point to Site VPN)

For roaming and external users, this lab uses **Point-to-Site VPN** to provide secure remote access into the Azure network. This simulates a common enterprise scenario where laptops or mobile users need to access cloud resources without being physically connected to the office network.

The Point-to-Site VPN connection connects the same Azure VPN Gateway deployed in the Hub VNet. Remote clients use the **Azure VPN Client** to establish an VPN tunnel into Azure. Once connected, the client receives access to the internal Azure address space and can reach permitted resources in the Hub and Spoke VNets through private IP addresses.

Access can be controlled through VPN configuration, routing, NSG rules and identity-based authentication.

![Architecture](/images/arch6.jpg)

## Secure Remote Administration (Bastion)

VMs in this lab are deployed without public IP addresses. Access is provided through **Azure Bastion**, which is placed in the **Hub VNet** and used as the central entry point for **RDP** and **SSH** access.

With this design, administrators do not need to expose **RDP port 3389** or **SSH port 22** to the public Internet. Instead, traffic enters through Azure Bastion and reaches the target VMs over private network paths.

The spoke virtual machines are accessed from the Hub through VNet peering. **Routing** and **NSG** rules are used to ensure that Bastion can reach the required private IP addresses while unnecessary inbound access is blocked. This ensur the safety of our private cloud environment compared with assigning public IP addresses directly to each VM.

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
