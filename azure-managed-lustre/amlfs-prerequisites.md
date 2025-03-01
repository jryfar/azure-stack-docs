---
title: Prerequisites for Azure Managed Lustre file systems (preview)
description: Network and storage prerequisites to complete before you create an Azure Managed Lustre file system.
ms.topic: overview
author: sethmanheim
ms.author: sethm 
ms.lastreviewed: 02/22/2023
ms.reviewer: mayabishop
ms.date: 02/09/2023

# Intent: As an IT Pro, I  need to understand network and storage requirements for using an Azure Managed Lustre file system, and I need to configure what I need.
# Keyword: 

---

# Azure Managed Lustre file system prerequisites (preview)

This article explains prerequisites that you must configure before creating an Azure Managed Lustre file system.

* [Network prerequisites](#network-prerequisites)
* [Blob integration prerequisites](#blob-integration-prerequisites-optional)
* [Azure Key Vault integration prerequisites (Optional)](#azure-key-vault-integration-requirements-optional)

[!INCLUDE [public-preview-disclaimer](includes/managed-lustre-preview-disclaimer.md)]

## Network prerequisites

Azure Managed Lustre file systems exist in a virtual network subnet. The subnet contains the Lustre Management Service (MGS) and handles all client interactions with the virtual Lustre cluster.

Each file system you create must have its own unique subnet. You can't move a file system from one network or subnet to another after you create the file system.

Azure Managed Lustre accepts only IPv4 addresses. IPv6 is not supported.

### Network size requirements

The size of subnet that you need depends on the size of the file system you create. The following table gives a rough estimate of the minimum subnet size for Azure Managed Lustre file systems of different sizes.

| Storage capacity     | Recommended CIDR prefix value |
|----------------------|-------------------------------|
| 4 TiB to 16 TiB      | /27 or larger                 |
| 20 TiB to 40 TiB     | /26 or larger                 |
| 44 TiB to 92 TiB     | /25 or larger                 |
| 96 TiB to 196 TiB    | /24 or larger                 |
| 200 TiB to 400 TiB   | /23 or larger                 |

#### Other network size considerations

When you plan your VNet and subnet, take into account the requirements for any other services you want to locate within the Azure Managed Lustre subnet or VNet. For example, consider the following factors.

* If you're using an Azure Kubernetes Service (AKS) cluster with your Azure Managed Lustre file system:

  * You can locate the AKS cluster in the same subnet as the managed Lustre system. In that case, you must provide enough IP addresses for the AKS nodes and pods in addition to the address space for the Lustre file system.

  * If you use more than one AKS cluster within the VNet, make sure the VNet has enough capacity for all resources in all of the clusters.
  
  To learn more about network strategies for Azure Managed Lustre and AKS, see [AKS subnet access](use-csi-driver-kubernetes.md#provide-subnet-access-between-aks-and-azure-managed-lustre).

* If you plan to use another resource to host your compute VMs in the same VNet, check the requirements for that process before creating the VNet and subnet for your Azure Managed Lustre system.

### Subnet access and permissions

By default, no specific changes need to be made to enable Azure Managed Lustre. If you have restricted network and/or security policies in your environment, the following should be considered:

| Access type | Required network settings |
|-------------|---------------------------|
| Create NICs permission |The file system must be able to create network interface cards (NICs) on its subnet. This setting is enabled by default. |
| DNS access  | Use the default Azure-based DNS server. |
| Azure Queue Storage service access |Azure Managed Lustre uses the Azure Queue Storage service to communicate configuration and state information. You can configure access in two ways:<br><br>**Option 1:** Add a private endpoint for Azure Storage to your subnet..<br><br>**Option 2:** Configure firewall rules to allow the following access:<br>- TCP port 443, for secure traffic to any host in the queue.core.windows.net domain (`*.queue.core.windows.net`)<br>- TCP port 80, for access to the certificate revocation list (CRL) and online certificate status protocol (OCSP) servers.<br><br>Contact your Azure Managed Lustre team if you need help with this requirement.|
|Azure cloud service access | Configure your network security group to permit the Azure Managed Lustre file system to access Azure cloud services from within the file system subnet.<br><br>Add an outbound security rule with the following properties:<br>- **Port**: Any<br>- **Protocol**: Any<br>- **Source**: Virtual Network<br>- **Destination**: "AzureCloud" service tag<br>- **Action**: Allow<br><br>For more information, see [Virtual network service tags](/azure/virtual-network/service-tags-overview).|
|Lustre network port access| Your network security group must allow inbound and outbound access on port 988 and ports 1019-1023.<br>The default rules `65000 AllowVnetInBound` and `65000 AllowVnetOutBound` meet this requirement.|
|Storage access |If you use Microsoft Azure Blob Storage integration with your Azure Managed Lustre file system, configure an Azure Storage endpoint so the file system can access the storage.
|Customer-managed encryption keys |If you use customer-managed encryption keys for your Azure Managed Lustre files, the file system must be able to access the associated Azure key vault.|

> [!NOTE]
> After you create your Azure Managed Lustre file system, several new network interfaces appear in the file system's resource group. Their names start with **amlfs-** and end with **-snic**. Don't change any settings on these interfaces. Specifically, leave the default value, **enabled**, for the **Accelerated networking** setting. Disabling accelerated networking on these network interfaces degrades your file system's performance.

## Blob integration prerequisites (optional)

If you plan to integrate your Azure Managed Lustre file system with Azure Blob Storage, complete the following prerequisites before you create your file system.

An integrated blob container can automatically import files to the Azure Managed Lustre system when you create the file system. You can then create archive jobs to export changed files from your Azure Managed Lustre file system to that container.

If you don't add an integrated blob container when you create your Lustre system, you can write your own client scripts or commands to move files between your Azure Managed Lustre file system and other storage.

When you plan blob integration for your file system, it's important to understand the differences in how hierarchical blob storage and non-hierarchical blob storage handle metadata. For more information, see [Understand hierarchical and non-hierarchical storage schemas](blob-integration.md#understand-hierarchical-and-non-hierarchical-storage-schemas).

To integrate Azure Blob Storage with your Azure Managed Lustre file system, you must create the following items before you create the file system:

* A storage account that meets the following requirements:

  * A compatible storage account type. See [Supported storage account types](#supported-storage-account-types) for more information.
  * A public endpoint. See [Storage account access](#storage-access-for-blob-integration) for more information.
  * Access roles that permit the Azure Managed Lustre system to modify data. See [Required access roles](#access-roles-for-blob-integration) for more information.

* A data container in the storage account that contains the files you want to use in the Azure Managed Lustre file system.

  You can add files to the file system later from clients. However, files added to the original blob container after you create the file system won't be imported to the Azure Managed Lustre file system.

* A second container for import/export logs in the storage account. You must store the logs in a different container from the data container.

### Supported storage account types

The following storage account types can be used with Azure Managed Lustre file systems.

| Storage account type  | Redundancy                          |
|-----------------------|-------------------------------------|
| Standard              | Locally redundant storage (LRS), geo-redundant storage (GRS)<br><br>Zone-redundant storage (ZRS), read-access-geo-redundant storage (RAGRS), geo-zone-redundant storage (GZRS), read-access-geo-zone-redundant storage (RA-GZRS) |
| Premium - Block blobs | LRS, ZRS |

For more information about storage account types, see [Types of storage accounts](/azure/storage/common/storage-account-overview#types-of-storage-accounts).

### Storage access for blob integration

Storage accounts used with an Azure Managed Lustre file system must have a public endpoint configured. However, you can restrict the endpoint to only accept traffic from the file system subnet. This configuration is needed because agents and copying tools are hosted in an infrastructure subscription, not within the customer's subscription.

> [!TIP]
> If you create the subnet before you create the storage account, you can configure restricted access when you create the storage account.

### Access roles for blob integration

Azure Managed Lustre needs authorization to access your storage account. Use [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/) to give the file system access to your blob storage.

A storage account owner must add these roles before creating the file system:

* [Storage Account Contributor](/azure/role-based-access-control/built-in-roles#storage-account-contributor)
* [Storage Blob Data Contributor](/azure/role-based-access-control/built-in-roles#storage-blob-data-contributor)

> [!IMPORTANT]
> You must add these roles before you create your Azure Managed Lustre file system. If the file system can't access your blob container, file system creation fails. Validation performed before the file system is created can't detect container access permission problems. It can take up to five minutes for the role settings to propagate through the Azure environment.

To add the roles for the service principal **HPC Cache Resource Provider**, do these steps:

1. Open your storage account, and select **Access control (IAM)** in the left navigation pane.

1. Select **Add** > **Add role assignment** to open the **Add role assignment** page.

1. Assign the role.

1. Then add the **HPC Cache Resource Provider** to that role.

   > [!TIP]
   > If you can't find the HPC Cache Resource Provider, search for **storagecache** instead. **storagecache Resource Provider** was the service principal name before general availability of the product.

1. Repeat steps 3 and 4 for to add each role.

For detailed steps, see [Assign Azure roles using the Azure portal](/azure/role-based-access-control/role-assignments-portal).

## Azure Key Vault integration requirements (optional)

If you want to create and manage the encryption keys that are used on your Azure Managed Lustre files, you can add customer-managed keys to an Azure Key Vault before or during file system creation. You can create a new key vault and key when you create the file system, but you should be familiar with these requirements ahead of time. To add a key vault and keys when you create the file system, you must have permissions to manage key vault access.

Using customer-managed keys is optional. All data stored in Azure is encrypted with Microsoft-managed keys by default. For more information, see [Azure storage encryption](/azure/storage/common/storage-service-encryption).

For more information about requirements for using customer-managed encryption keys with an Azure Managed Lustre file system, including prerequisites, see [Use customer-managed encryption keys with Azure Managed Lustre](customer-managed-encryption-keys.md).

If you plan to use customer-managed encryption keys with your Azure Managed Lustre file system, complete the following prerequisites:

* Create an Azure key vault that meets the following requirements, either before or during file system creation:

  * The key vault must be in the same subscription and Azure region as the Azure Managed Lustre file system.
  * Enable **soft delete** and **purge protection** on the key vault.
  * Ensure there is network access between the key vault and the Azure Managed Lustre file system. By default, the access exists.
  * Keys must be 2048-bit RSA keys.

  For more Azure Key Vault requirements, see [Use customer-managed encryption keys with Azure Managed Lustre](customer-managed-encryption-keys.md).

* Create a user-assigned managed identity for the file system to use when accessing the key vault. For more information, see [What are managed identities for Azure resources?](/azure/active-directory/managed-identities-azure-resources/overview)

* Assign the [Key Vault contributor role](/azure/role-based-access-control/built-in-roles#key-vault-contributor) to the person who will create the Azure Managed Lustre file system. The Key vault contributor role is required in order to manage key vault access. For more information, see [Use customer-managed encryption keys with Azure Managed Lustre](customer-managed-encryption-keys.md).

## Next steps

* [Create an Azure Managed Lustre file system in the Azure portal](create-file-system-portal.md)
