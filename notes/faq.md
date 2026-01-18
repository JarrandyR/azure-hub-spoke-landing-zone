# FAQ

## What is this project?

This repository demonstrates a **hub-and-spoke virtual network topology in Microsoft Azure**, aligned with **Azure Landing Zone networking principles**.  
It focuses on building a **secure, scalable network foundation** using Azure-native services and Azure CLI.

---

## Is this a full Azure Landing Zone implementation?

No.

This project is **Landing Zone–aligned**, not a full enterprise Landing Zone.  
It intentionally focuses on the **networking layer only**, specifically:

- Resource group separation
- Hub-and-spoke VNet design
- Private VNet peering
- Subnet-level security controls using NSGs

It does **not** include:
- Management Groups
- Azure Policy
- Identity subscriptions
- Centralized logging or monitoring
- Azure Firewall or NVAs

---

## Why use a hub-and-spoke topology?

Hub-and-spoke is a widely adopted Azure networking pattern because it:

- Separates shared platform networking from workloads
- Simplifies network governance and scaling
- Enables centralized inspection and routing
- Supports future expansion (additional spokes, firewalls, hybrid connectivity)

This lab demonstrates the **foundational pattern** before adding advanced services.

---

## Why is the hub network mostly empty?

In this lab, the hub represents a **platform network boundary**, not a service-heavy hub.

The focus is on:
- Address space planning
- Connectivity
- Network segmentation

In production, the hub would typically host:
- Azure Firewall or NVA
- VPN / ExpressRoute gateways
- Shared services (DNS, jump hosts, etc.)

---

## Why are NSGs used instead of Azure Firewall?

NSGs are used to demonstrate **baseline workload security** without adding cost or architectural complexity.

The spoke subnet NSG:
- Allows inbound traffic from the hub
- Denies inbound traffic from the Internet

This establishes a **default-secure posture** while keeping the lab lightweight.

---

## Why Azure CLI instead of the Azure Portal?

Azure CLI was chosen because it is:

- Scriptable and repeatable
- Automation-friendly
- Easier to audit and version-control
- More representative of real-world engineering workflows

All resources can be recreated consistently from the documented commands.

---

## Can this design be extended?

Yes. Common next steps include:

- Adding Azure Firewall or an NVA in the hub
- Introducing UDRs for centralized traffic inspection
- Adding multiple spoke VNets for different workloads
- Converting the deployment to Bicep or Terraform
- Integrating monitoring and logging

This project is designed as a **foundation**, not an endpoint.

---

## Who is this project for?

- Cloud engineers learning Azure networking
- Security-focused engineers building network baselines
- Students or professionals building a portfolio
- Anyone wanting a clean, reproducible hub-and-spoke example

---

## Does this project incur Azure costs?

Yes, minimal costs may be incurred while resources exist.

All resources are safe to delete after validation using:
```bash
az group delete --name <resource-group> --yes --no-wait
