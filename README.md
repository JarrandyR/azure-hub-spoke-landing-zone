# Azure Hub-and-Spoke Network Baseline (Landing Zone–Aligned)

## Overview
This project demonstrates the design and implementation of a **hub-and-spoke virtual network topology in Microsoft Azure**, aligned with **Azure Landing Zone networking principles**.

The goal of this lab is to establish a **secure, scalable network foundation** that separates shared platform networking from workload networks, using Azure-native services and a terminal-first workflow.

This repository documents the full build using **Azure CLI**, includes validation evidence, and provides clear steps so others can reproduce the environment.

---

## Architecture Overview

```mermaid
flowchart LR
    Internet((Internet))

    subgraph Hub["Hub VNet (10.0.0.0/16)"]
        HS["snet-shared\n10.0.0.0/24"]
    end

    subgraph Spoke["Spoke VNet (10.1.0.0/16)"]
        AS["snet-app\n10.1.0.0/24"]
        NSG["NSG:\nAllow Hub\nDeny Internet"]
    end

    Internet -.->|Blocked| AS
    HS <--> AS
    NSG --> AS
    ```
    
