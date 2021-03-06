---
title: Protect your Honolulu-managed Hyper-V Virtual Machines with Azure Site Recovery
description: "Protect VMs from disaster with Honolulu managed ASR"
ms.prod: windows-server-threshold
ms.reviewer: na
ms.suite: na
ms.technology: Honolulu
ms.tgt_pltfrm: na
ms.topic:
author: coreyp-at-msft
ms.author: coreyp
manager: dongill
ms.date: 12/01/2017
---
# Protect your Honolulu-managed Hyper-V Virtual Machines with Azure Site Recovery 

>Applies To: Windows Server (Semi-Annual Channel), Windows Server 2016, Windows Server 2012 R2, Windows Server 2012 and Windows 10

Project Honolulu streamlines the process of replicating your virtual machines on your Hyper-V servers or clusters, making it easier to leverage the power of Azure from your own datacenter. To automate setup, you can connect the Honolulu gateway to Azure.

## What is Azure Site Recovery and how does it work with Honolulu? 

**Azure Site Recovery** is an Azure service that replicates workloads running on VMs so that your business-critical infrastructure is protected in case of a disaster.  [Learn more about Azure Site Recovery](https://docs.microsoft.com/en-us/azure/site-recovery/site-recovery-overview).

Azure Site Recovery consists of two components: **replication** and **failover**. The replication portion protects your VMs in case of disaster by replicating the target VM's VHD to an Azure storage account. You can then failover these VMs and run them in Azure in the event of a disaster. You can also perform a test failover without impacting your primary VMs to test the recovery process in Azure.

Completing setup for the replication component alone is sufficient to protect your VM in the case of disaster. However, you will not be able to start the VM in Azure until you have configured the failover portion. The failover portion can be setup at the time you wish to failover to an Azure VM, and is not required as part of initial setup. If the host server goes down and you haven't yet configured the failover component, you can configure it at that time and access the workloads of the protected VM. However, it is a good practice to configure the failover related settings before a disaster.
 
## In this guide

This guide outlines the steps to configure replication settings and create a recovery plan from within the Azure portal, enabling Honolulu to start VM replication and protect your VMs.

## Prerequisites and planning

- The target servers hosting the VMs you wish to protect must have Internet access to replicate to Azure.
- [Connect your Honolulu gateway to Azure](https://aka.ms/HonoluluGatewayAzureSetup).
- [Review the capacity planning tool to evaluate the requirements for successful replication and failover](https://docs.microsoft.com/en-us/azure/site-recovery/hyper-v-site-walkthrough-capacity).

## Step 1: Install the ASR agent onto your target host and setup ASR requirements in Azure

> [!NOTE] 
> You need to do this step once per host server or cluster containing VMs targeted for protection.

1. Navigate to the server or cluster hosting VMs you wish to protect (either with Server Manager or Hyper-Converged Cluster Manager).
2. Go to **Virtual Machines** > **Inventory**.
3. Select any VM (this doesn't need to be the VM you want to protect).
4. Select **More** > **Protect VM**.
5. Login to your Azure Account in the popup dialog.
6. Enter the required parameters:

 - **Subscription:** The Azure subscription you want to use for replication of VMs on this host.
 - **Location:** The Azure region where the ASR resources should be created.
 - **Storage Account:** The storage account where replicated VM workloads on this host will be saved.
 - **Vault:** Choose a name for the Azure Site Recovery vault for protected VMs on this host.

7.  Select **Setup ASR**.
8.  Wait until you see the notification: **Site Recovery Setting Completed**.
 
This could take up to 10 minutes. You can watch the progress by going to **Notifications** (the bell icon at the top right).

>[!NOTE]
> This step automatically installs the ASR agent onto the target server or nodes (if configuring on a cluster), creates a **Resource Group** with the **Storage Account** and **Vault** specified, in the **Location** specified. This will also register the target host with the ASR service and configure a default replication policy.

## Step 2: Select Virtual Machines to protect

1. Navigate back to the server or cluster you configured in step 2 above, and go to **Virtual Machines > Inventory**.
2. Select the VM you want to protect.
3. Select **More** > **Protect VM**.
4. Review the [capacity requirements for protecting the VM](https://docs.microsoft.com/en-us/azure/site-recovery/site-recovery-capacity-planner).

    If you want to use a premium storage account, [create one in the Azure portal](https://docs.microsoft.com/en-us/azure/storage/common/storage-premium-storage). The **Create New** option provided in the Honolulu pane will create a standard storage account.

5. Enter the name of the **Storage Account** to use for this VM's replication and select **Protect VM**. This step enables replication for the selected Virtual Machine. 

6. ASR will start replication. Replication is completed and the VM is protected when the value in the **Protected** column of the **Virtual Machine Inventory** grid changes to **Yes**. This can take several minutes.  

## Step 3: Configure and run a test failover in the Azure Portal

 Although this step does not need to be completed when starting VM replication (the VM will already be protected with just replication), we recommend you configure failover settings when setting up Azure Site Recovery. If you want to prepare for failover to an Azure VM, complete the following steps:

1. [Set up an Azure network](https://docs.microsoft.com/en-us/azure/site-recovery/hyper-v-site-walkthrough-prepare-azure) the failed-over VM will attach to this VNET. Note that the other steps listed in the linked page are completed automatically by Honolulu;  you only need to set up the Azure network.

2. [Run a test failover](https://docs.microsoft.com/en-us/azure/site-recovery/hyper-v-site-walkthrough-test-failover).

## Step 4: Create Recovery plans

**Recovery Plan** is a feature in Azure Site Recovery that lets you failover and recover an entire application comprising a collection of VMs. While it is possible to recover protected VMs individually, by adding the VMs comprising an application to a recovery plan, you'll be able to failover the entire application through the recovery plan. You can also use the test failover feature of Recovery Plan to test the recovery of the application. Recovery Plan lets you group VMs, sequence the order in which they should be brought up during a failover, and automate additional steps to be performed as part of the recovery process. Once you've protected your VMs, you can go to the Azure Site Recovery vault in the Azure portal and create recovery plans for these VMs. [Learn more about recovery plans](https://docs.microsoft.com/en-us/azure/site-recovery/site-recovery-create-recovery-plans).

## Monitoring replicated VMs in Azure ##

To verify there are no failures in server registration, go to the **Azure portal** > **All resources** > **Recovery Services Vault**  (the one you specified in Step 2) > **Jobs** > **Site Recovery Jobs**.

You can monitor VM replication by going to the **Recovery Services Vault** > **Replicated Items**.

To see all servers that are registered to the vault, go to **Recovery Services Vault** > **Site Recovery Infrastructure** > **Hyper-V hosts** (under the Hyper-V sites section).

## Known issue ##

When registering ASR with a cluster, if a node fails to install ASR or register to the ASR service, your VMs may not be protected. Verify that all nodes in the cluster are registered in the Azure portal by going to the **Recovery Services vault** > **Jobs** > **Site Recovery Jobs**.
