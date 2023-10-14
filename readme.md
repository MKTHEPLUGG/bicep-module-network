# Azure Bicep Network & NSG Deployment Module

This repository provides an Azure Bicep module that facilitates the deployment of a network and NSG (Network Security Group) in Azure. 

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Module Description](#module-description)
  - [Allowed Inputs](#allowed-inputs)
  - [Resources](#resources)
- [GitHub Actions](#github-actions)
  - [Tag a Git Version](#tag-a-git-version)
  - [Publish Module](#publish-module)
- [Usage](#usage)
- [Contributors](#contributors)

---

## Overview

The Bicep module contained within this repository will deploy a virtual network along with a network security group (NSG) in a specified Azure location. Additionally, the module leverages GitHub Actions to tag a git version and to publish the Bicep module.

---

## Prerequisites

- An Azure account with necessary permissions for deploying resources.
- GitHub repository to implement CI/CD using GitHub Actions.
- Familiarity with Azure Bicep language.

---

## Module Description

### Allowed Inputs

- `location`: Specifies the Azure location where resources are to be deployed.
  - Allowed values: `westeurope`, `northeurope`.
  
- `environment`: Specifies the deployment environment.
  - Allowed values: `integration`, `acceptance`, `production`.

Other parameters include:
- `purpose`: Describes the purpose of the network.
- `addressPrefixes`: Array of address prefixes for the virtual network.
- `subnetPrefix`: Address prefix for the subnet.
- `gitRepo`: Repository name.
- `gitRef`: Git reference (usually the commit or tag).

### Resources

1. **Network Security Group (NSG)**: This resource is named based on the `subnetName` and is tagged with the git repository, git reference, and environment details. It contains security rules, like allowing SSH.

2. **Virtual Network (VNET)**: This resource is named based on the environment and location. It contains address spaces defined by the `addressPrefixes` parameter and a subnet defined by the `subnetPrefix` parameter.

---

## GitHub Actions

### Tag a Git Version

This action is designed to tag the GitHub repository with a specific git version. This is especially useful for versioning releases or specific states of your codebase.

**Triggers**:
- On push to the `main` branch.
- Manual trigger through the GitHub UI (`workflow_dispatch`).

### Publish Module

This action publishes the Bicep module to a specified target. It's triggered when a new tag is pushed to the repository, ensuring that the Bicep module is published with the corresponding version.

**Triggers**:
- On push of a new tag.
- Manual trigger through the GitHub UI (`workflow_dispatch`).

---

## Usage

To deploy the network and NSG using this Bicep module, ensure that you:

1. Setup the necessary secrets in your GitHub repository for Azure login (`AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`).
2. Use the provided GitHub Actions or integrate them into your existing CI/CD processes.

---

## Contributors

- Michiel Van Haegenborgh

For any further questions or contributions, please open an issue or submit a pull request.

---

Note: This README provides a comprehensive guide for users to understand and use the Bicep module and associated GitHub Actions. Depending on the actual content and structure of the repository, some minor adjustments might be necessary.





## Bicep networking module

this module is used to create networks within azure. Pipelines included to auto deploy via Github, always needs to be in .github/Workflows

this is our main example of RD CICD Course





### Usefull shortcuts

PRESS `SHIFT + .` to open code in browser !!!
