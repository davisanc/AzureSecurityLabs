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

### 6.2 - Applying VM disk encryption

You can enable encryption by using a template, PowerShell cmdlets, or CLI commands. The following sections explain in detail how to enable Azure Disk Encryption.

*Important: It is mandatory to snapshot and/or backup a managed disk based VM instance outside of, and prior to enabling Azure Disk Encryption. A snapshot of the managed disk can be taken from the portal, or Azure Backup can be used. Backups ensure that a recovery option is possible in the case of any unexpected failure during encryption.*

#### Run a prerequisite script to encrypt the data disk of your VM(s)

1. Make sure you have PowerShell version 6 installed on your local machine.
2. Verify the installed versions of the AzureRM module. The AzureRM module version needs to be 6.0.0 or higher.

    ```PowerShell
    Get-Module AzureRM -ListAvailable | Select-Object -Property Name,Version,Path
    ```

3. If needed, update the Azure PowerShell module. Using the latest AzureRM module version is recommended (This will require administrative rights to the PowerShell session).

    ```PowerShell
    Update-Module -Name AzureRM
    ```

#### Enable encryption on existing or running VMs with Azure CLI

There are some prerequistes to check before enabling disk encryption. They can be found here:

https://docs.microsoft.com/en-us/azure/security/azure-security-disk-encryption-prerequisites

We will need an Azure Active Directory (AAD) application that will be used to write secrets to KeyVault as an authentication step. Also, we need a secret of the AAD application that was created on the earlier step. Recommendation is to run through the pre-req powershell script that handles this

I will use the following names for the AAD App name and client secret. Running through the script, this App will be registered with AAD and will be authorized to use KeyVault. This client secret will be written in KeyVault

```text
aadAppName: keyvault-dasanc-app
aadClientSecret: dasancsec
```

On the portal go to **Azure Active Directory, App Registrations**, click **New Application Registration** and create the App that will write the secret to KeyVault

#### Encrypt a running VM

##### Azure CLI

```bash
az vm encryption enable -g Sec-Foundation-MTC --name "sql-vm1" --disk-encryption-keyvault "KeyVault-MTC-Sec" --aad-client-id "David Sanchez" --aad-client-secret "<your-secret>"--volume-type ALL
```

##### Powershell

Powershell pre-req script: you can download the 'DiskEncryption.ps' file available here

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

Using the pre-req script you get the final result for the SQL VM...

![sql vm encrypted](/images/sql-vm-encrypted.PNG)

<< [Back to home page](/README.md)

<< [Previous Lab](lab-05-security-centre.md) . . . . . [Next Lab](lab-07-site-to-site-vpn.md) >>