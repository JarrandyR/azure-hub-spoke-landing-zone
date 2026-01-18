# Azure Hub-and-Spoke Network Architecture

This document describes the **network architecture** implemented in this repository.  
The goal is to clearly explain **what was built**, **how traffic flows**, and **why this design is used**, without requiring prior knowledge of the lab steps.

This architecture follows **Azure Landing Zone networking concepts** by separating shared platform networking from application workloads.

---

## High-Level Design

The environment uses a **hub-and-spoke virtual network topology**:

- The **Hub VNet** acts as the centralized platform network.
- The **Spoke VNet** represents an isolated workload network.
- Connectivity between the two is provided through **private VNet peering**.
- Security is enforced at the **spoke subnet level** using a Network Security Group (NSG).

This pattern is commonly used in real-world Azure environments to improve security, scalability, and operational clarity.

---

## Logical Architecture Diagram

```mermaid
flowchart TB
    Internet((Internet))

    subgraph Hub["Hub VNet (10.0.0.0/16)"]
        HubSubnet["snet-shared (10.0.0.0/24)"]
    end

    subgraph Spoke["Spoke VNet (10.1.0.0/16)"]
        AppSubnet["snet-app (10.1.0.0/24)"]
        NSG["NSG\nAllow Hub\nDeny Internet"]
    end

    Internet -.->|Blocked| AppSubnet
    HubSubnet <-->|VNet Peering| AppSubnet
    NSG --> AppSubnet






    Component Explanation
1. Internet

Represents untrusted external traffic.
Inbound traffic originating from the Internet is not allowed to reach the application subnet.

2. Hub Virtual Network (Platform Layer)

Address Space: 10.0.0.0/16
Subnet: snet-shared (10.0.0.0/24)

Purpose:

Centralized networking layer

Trusted connectivity source for workloads

Future location for shared services such as:

Azure Firewall or NVA

VPN / ExpressRoute Gateway

Bastion

Centralized DNS

No application workloads are deployed in the hub network.

3. Spoke Virtual Network (Workload Layer)

Address Space: 10.1.0.0/16
Subnet: snet-app (10.1.0.0/24)

Purpose:

Hosts application workloads

Isolated from direct Internet access

Communicates privately with the hub network

This subnet represents where virtual machines, containers, or platform services would normally be deployed.

4. VNet Peering (Hub ↔ Spoke)

Type: Bidirectional Azure VNet peering
Traffic Path: Microsoft backbone network

Why it matters:

Enables private, low-latency communication

No public IP exposure

Each peering is explicit and non-transitive

Supports scalable growth by adding additional spokes

Traffic can flow securely between the hub and spoke without traversing the public Internet.

5. Network Security Group (NSG)

Applied To: Spoke subnet (snet-app)

Inbound Rules:

✅ Allow traffic from the hub address space (10.0.0.0/16)

❌ Deny traffic originating from the Internet

Purpose:

Enforces security at the workload boundary

Demonstrates least-privilege network access

Prevents direct external access to applications

This mirrors common enterprise security baselines for workload networks.

Traffic Flow Summary
Source	Destination	Result	Reason
Hub VNet	Spoke Subnet	Allowed	Trusted internal traffic
Internet	Spoke Subnet	Denied	Blocked by NSG
Spoke Subnet	Hub VNet	Allowed	Private peering
Landing Zone Alignment

This architecture aligns with Azure Landing Zone networking principles by:

Separating platform and workload responsibilities

Supporting horizontal scalability with additional spokes

Using private connectivity by default

Enforcing defense-in-depth at the subnet level

This implementation intentionally excludes governance features such as management groups, Azure Policy, and centralized logging to keep the focus on core network architecture.