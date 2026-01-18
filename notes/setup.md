 Setup & Prerequisites

This lab assumes a local workstation with Azure CLI access and permissions to create networking resources.

## Required Tools

- Azure CLI (v2.x or later)
- PowerShell (Windows) or Bash (macOS/Linux)
- Git
- Azure subscription with Owner or Contributor permissions

## Azure Requirements

- Active Azure subscription
- Ability to create:
  - Resource Groups
  - Virtual Networks
  - Subnets
  - Network Security Groups
  - VNet Peerings

## Authentication

You must be authenticated to Azure before starting:

```bash
az login

Verify the active subscription:

az account show --output table