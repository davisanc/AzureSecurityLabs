# Lab 6 - Storage Security

## Encryption at Rest and applying disk encryption to a running VM

Having looked at Azure Security Center we can see recommendations to apply disk encryption to our VMs. This can be done with the help of Azure Key Vault.

Full details about Azure Key Vault can be found here: [https://azure.microsoft.com/en-us/services/key-vault/](https://azure.microsoft.com/en-us/services/key-vault/)

For Azure Disk encryption to work, the Key Vault and the VMs must be co-located in the same Azure region and subscription.

### 6.1 - Create the Azure Key Vault

Create a new Key Vault from the portal following the steps in the image below, or create a Key Vault using this CLI command:

```bash
az keyvault create --name <your-keyvault-name> --resource-group <resource-group-name> --sku standard
```

![create keyv](/images/Create-KeyVault.PNG)

### 6.2 - Prerequisites to go before running encryption

You can enable encryption by using a template, PowerShell cmdlets, or CLI commands. The following sections explain in detail how to enable Azure Disk Encryption.

*Important: It is mandatory to snapshot and/or backup a managed disk based VM instance outside of, and prior to enabling Azure Disk Encryption. A snapshot of the managed disk can be taken from the portal, or Azure Backup can be used. Backups ensure that a recovery option is possible in the case of any unexpected failure during encryption.*

There are some prerequistes to check before enabling disk encryption. They can be found here:
https://docs.microsoft.com/en-us/azure/security/azure-security-disk-encryption-prerequisites

We have summarized them for you here. Alternatively, you can run a script that will go through all the following comamnds for you

**IMPORTAN: If you prefer to do this one by one, go through the following steps. If you prefer to go with the scritpt, jumpt to 9.4 of this lab**

**On Powershell**

1. Make sure you have PowerShell version 6 installed on your local machine.
2. Verify the installed versions of the AzureRM module. The AzureRM module version needs to be 6.0.0 or higher.

    ```PowerShell
    Get-Module AzureRM -ListAvailable | Select-Object -Property Name,Version,Path
    ```

3. If needed, update the Azure PowerShell module. Using the latest AzureRM module version is recommended (This will require administrative rights to the PowerShell session).

    ```PowerShell
    Update-Module -Name AzureRM
    ```
4. Install the Azure Active Directory PowerShell module

    ```
    Install-Module AzureAD
    ```

5. Verify the installed versions of the module

    ```
    Get-Module AzureAD -ListAvailable | Select-Object -Property Name,Version,Path
    ```
6. Set up an Azure AD app and service principal
    
    We need an Azure Active Directory (AAD) application that will be used to write secrets to KeyVault as an authentiation step. Also,  we need a secret of the AAD application that was created on the earlier step. Recommendation is to run through the powershell script that handles this
    
As a reference, I will use the following names for the AAD App name and client secret. Running through the script, this App will be registered with AAD and will be authorized to use KeyVault. This client secret will be written in KeyVault

```text
aadAppName: keyvault-dasanc-app
aadClientSecret: dasancsec
```
   
   On Az CLI:
    
    ```bash
    az ad sp create-for-rbac --name "ServicePrincipalName" --password "My-AAD-client-secret" --skip-assignment
    ```
    
    *Note:The appId returned is the Azure AD ClientID used in other commands. It's also the SPN you'll use for az keyvault set-policy. The password is the client secret that you should use later to enable Azure Disk Encryption. Safeguard the Azure AD client secret appropriately.*
    
7. Set the key vault access policy for the Azure AD app with Azure CLI

    ```
    az keyvault set-policy --name "MySecureVault" --spn "<spn created with CLI/the Azure AD ClientID>" --key-permissions wrapKey -- secret-permissions set
    ```
8. Set key vault advanced access policies

The Azure platform needs access to the encryption keys or secrets in your key vault to make them available to the VM for booting and decrypting the volumes. Enable disk encryption on the key vault or deployments will fail.

Set key vault advanced access policies through the Azure portal
Select your keyvault, go to Access Policies, and Click to show advanced access policies.
Select the box labeled Enable access to Azure Disk Encryption for volume encryption.
Select Enable access to Azure Virtual Machines for deployment and/or Enable Access to Azure Resource Manager for template deployment, if needed.
Click Save.

![image of key vault advanced](/images/keyvault-advancedset.PNG)

### 6.3 Enable encryption on existing or running VMs with Azure CLI

**Azure CLI**

```bash
az vm encryption enable -g Sec-Foundation-MTC --name "sql-vm1" --disk-encryption-keyvault "KeyVault-MTC-Sec" --aad-client-id "David Sanchez" --aad-client-secret "<your-secret>"--volume-type ALL
```


**Powershell**

```PowerShell
$rgName = 'MySecureRg';
 $vmName = 'MySecureVM';
 $KeyVaultName = 'MySecureVault';
 $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
 $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
 $KeyVaultResourceId = $KeyVault.ResourceId;

 Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId;
```

*Note: you can also run disk encryption with key encryption key (out of scope of this lab)*

**Finally, verify the disks are encrypted**
```
az vm encryption show --name "MySecureVM" --resource-group "MySecureRg"
```

![image od disk-enc](/images/disk-enc.PNG)


### 6.4 Use the script

You can also use the script, that runs all the previous commands for you
Use the AzureDiskEncryptionPreRequisiteSetup.ps1 on this repository, and run it on Powershell

The script will require the Resource Group name, KeyVault name, Location, Subscription ID, and AAD App name and client secret
We recommend using PowerShell ISE 

Full instructions are here: https://docs.microsoft.com/en-us/azure/security/quick-encrypt-vm-powershell

This shoudd be the final result of the script

![sql vm encrypted](/images/sql-vm-encrypted.PNG)

Verify the disks are encrypted: To check on the encryption status of a IaaS VM, use the Get-AzureRmVmDiskEncryptionStatus cmdlet

```
Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName <resource-group-name> -VMName <vm-name>

```
Finally, if we get back to Azure Security Center, on the recommendations panel we will no longer see the recommendation on the SQL VM to apply disk encryption (it will need some time to refresh the new status)


<< [Back to home page](/README.md)

<< [Previous Lab](lab-05-security-centre.md) . . . . . [Next Lab](lab-07-site-to-site-vpn.md) >>
