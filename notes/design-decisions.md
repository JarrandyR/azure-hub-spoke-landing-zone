# Design Decisions

## Why Hub-and-Spoke
Hub-and-spoke is a common Azure networking pattern where shared networking/services live in a central hub,
and workloads live in isolated spokes. This improves separation of concerns and supports scaling.

## Why VNet Peering
VNet peering provides private connectivity between VNets over Microsoft’s backbone. It is not transitive,
so hub-and-spoke is designed intentionally with explicit peerings.

## Why a Basic NSG Baseline on the Spoke
The spoke represents a workload boundary. Applying an NSG baseline demonstrates basic security posture:
permit trusted internal flows and restrict direct Internet inbound paths.

## Why This Is “Landing Zone–Aligned” but Not a Full Landing Zone
This lab aligns to landing zone networking concepts (separation of shared vs workload networking),
but does not implement full governance (management groups, policy-as-code, multi-subscription strategy).

Architecture Highlights:

Hub VNet hosts shared networking components

Spoke VNet represents isolated workloads

VNet Peering enables private connectivity over Azure backbone

NSG on Spoke Subnet enforces inbound security controls

Technologies Used:

Microsoft Azure

Azure Virtual Networks (VNet)

Azure VNet Peering

Network Security Groups (NSG)

Azure CLI

GitHub (documentation & version control)


| Component       | Purpose                                  |
| --------------- | ---------------------------------------- |
| Hub VNet        | Centralized shared networking foundation |
| Spoke VNet      | Isolated workload network                |
| VNet Peering    | Private, low-latency connectivity        |
| NSG             | Inbound traffic control for workloads    |
| Resource Groups | Lifecycle and cost separation            |


| Resource     | CIDR        |
| ------------ | ----------- |
| Hub VNet     | 10.0.0.0/16 |
| Hub Subnet   | 10.0.0.0/24 |
| Spoke VNet   | 10.1.0.0/16 |
| Spoke Subnet | 10.1.0.0/24 |
