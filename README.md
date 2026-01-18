# Azure Hub-and-Spoke Network Baseline (Landing Zone–Aligned)

This repository documents the implementation of a **hub-and-spoke virtual network topology in Microsoft Azure**, aligned with **Azure Landing Zone networking principles**. The lab is built entirely using **Azure CLI** and demonstrates foundational platform and workload network separation, private connectivity, and subnet-level security controls.

The structure of this guide allows readers to follow the build sequentially while reviewing validation evidence in the `/screenshots` directory.

---

## Environment Overview

- **Region:** East US
- **Hub Network (Platform Layer)**
  - VNet: `vnet-hub-eastus` — `10.0.0.0/16`
  - Subnet: `snet-shared` — `10.0.0.0/24`
- **Spoke Network (Workload Layer)**
  - VNet: `vnet-spoke-app-eastus` — `10.1.0.0/16`
  - Subnet: `snet-app` — `10.1.0.0/24`
- **Connectivity**
  - Bidirectional VNet peering (hub ↔ spoke)
- **Security**
  - Network Security Group applied to the spoke subnet

---

## Prerequisites

- Azure CLI installed
- PowerShell terminal
- Active Azure subscription

---

## Step 01 — Authenticate and verify subscription

Authenticate to Azure and confirm an active subscription context.

```powershell
az login
az account show --output table
Step 02 — Create resource groups
Separate resource groups are created to isolate platform networking (hub) from workload networking (spoke).

$LOCATION="eastus"
$RG_HUB="rg-net-hub-eastus"
$RG_SPOKE="rg-net-spoke-eastus"

az group create --name $RG_HUB --location $LOCATION
az group create --name $RG_SPOKE --location $LOCATION
Optional verification:

az group list --query "[?starts_with(name,'rg-net-')].[name,location]" -o table
Step 03 — Create the hub virtual network
The hub network acts as the shared platform connectivity layer.

$VNET_HUB="vnet-hub-eastus"
$HUB_CIDR="10.0.0.0/16"
$HUB_SUBNET_NAME="snet-shared"
$HUB_SUBNET_CIDR="10.0.0.0/24"

az network vnet create `
  --resource-group $RG_HUB `
  --name $VNET_HUB `
  --location $LOCATION `
  --address-prefixes $HUB_CIDR

az network vnet subnet create `
  --resource-group $RG_HUB `
  --vnet-name $VNET_HUB `
  --name $HUB_SUBNET_NAME `
  --address-prefixes $HUB_SUBNET_CIDR
Step 04 — Create the spoke virtual network
The spoke network represents an isolated workload environment.

$VNET_SPOKE="vnet-spoke-app-eastus"
$SPOKE_CIDR="10.1.0.0/16"
$SPOKE_SUBNET_NAME="snet-app"
$SPOKE_SUBNET_CIDR="10.1.0.0/24"

az network vnet create `
  --resource-group $RG_SPOKE `
  --name $VNET_SPOKE `
  --location $LOCATION `
  --address-prefixes $SPOKE_CIDR

az network vnet subnet create `
  --resource-group $RG_SPOKE `
  --vnet-name $VNET_SPOKE `
  --name $SPOKE_SUBNET_NAME `
  --address-prefixes $SPOKE_SUBNET_CIDR
Step 05 — Configure VNet peering (hub ↔ spoke)
Private connectivity is enabled using bidirectional VNet peering.

$SUB_ID = az account show --query id -o tsv

az network vnet peering create `
  --name peer-hub-to-spoke `
  --resource-group $RG_HUB `
  --vnet-name $VNET_HUB `
  --remote-vnet "/subscriptions/$SUB_ID/resourceGroups/$RG_SPOKE/providers/Microsoft.Network/virtualNetworks/$VNET_SPOKE" `
  --allow-vnet-access

az network vnet peering create `
  --name peer-spoke-to-hub `
  --resource-group $RG_SPOKE `
  --vnet-name $VNET_SPOKE `
  --remote-vnet "/subscriptions/$SUB_ID/resourceGroups/$RG_HUB/providers/Microsoft.Network/virtualNetworks/$VNET_HUB" `
  --allow-vnet-access
Verification:

az network vnet peering list --resource-group $RG_HUB --vnet-name $VNET_HUB -o table
az network vnet peering list --resource-group $RG_SPOKE --vnet-name $VNET_SPOKE -o table
Step 06 — Apply NSG to the spoke subnet
A Network Security Group is used to enforce workload-level security.

$NSG_SPOKE="nsg-spoke-app-eastus"

az network nsg create `
  --resource-group $RG_SPOKE `
  --name $NSG_SPOKE `
  --location $LOCATION

az network nsg rule create `
  --resource-group $RG_SPOKE `
  --nsg-name $NSG_SPOKE `
  --name Allow-Hub-In `
  --priority 100 `
  --direction Inbound `
  --access Allow `
  --protocol "*" `
  --source-address-prefixes "10.0.0.0/16" `
  --source-port-ranges "*" `
  --destination-address-prefixes "*" `
  --destination-port-ranges "*"

az network nsg rule create `
  --resource-group $RG_SPOKE `
  --nsg-name $NSG_SPOKE `
  --name Deny-Internet-In `
  --priority 200 `
  --direction Inbound `
  --access Deny `
  --protocol "*" `
  --source-address-prefixes "Internet" `
  --source-port-ranges "*" `
  --destination-address-prefixes "*" `
  --destination-port-ranges "*"

az network vnet subnet update `
  --resource-group $RG_SPOKE `
  --vnet-name $VNET_SPOKE `
  --name $SPOKE_SUBNET_NAME `
  --network-security-group $NSG_SPOKE
Step 07 — Final validation
Confirm peering connectivity and subnet security configuration.

az network vnet peering list --resource-group $RG_HUB --vnet-name $VNET_HUB -o table
az network vnet peering list --resource-group $RG_SPOKE --vnet-name $VNET_SPOKE -o table

az network vnet subnet show `
  --resource-group $RG_SPOKE `
  --vnet-name $VNET_SPOKE `
  --name $SPOKE_SUBNET_NAME `
  --query "{Subnet:name,Prefix:addressPrefix,NSG:networkSecurityGroup.id}" `
  -o table
Design Documentation
Network architecture explanation: architecture/architecture.md

Design rationale: notes/design-decisions.md

Cleanup
az group delete --name $RG_HUB --yes --no-wait
az group delete --name $RG_SPOKE --yes --no-wait
