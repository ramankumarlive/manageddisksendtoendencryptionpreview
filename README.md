# End to end encryption of VM/VMSS disks in preview

This repository contains samples templates for enabling the preview of the following encryption:

1. Encryption at rest for temp disks
2. Encryption at rest for cache of OS and data Managed Disks 
3. Encryption of data-in-transit of OS and data Managed Disks 

The above encryption features can be enabled by setting a new property EncryptionAtHost under securityProfile of VMs/VMSSs using the API version *2020-06-01* and above.

"securityProfile": { "encryptionAtHost": "true" }

## Prerequisite
You must enable the feature for your subscription before you use the EncryptionAtHost property for your VM/VMSS. Please follow the steps below to enable the feature for your subscription:

1. Execute the following command to register the feature for your subscription 
```PowerShell
 Register-AzProviderFeature -FeatureName "EncryptionAtHost" -ProviderNamespace "Microsoft.Compute" 
```
2. Please check that the registration state is Registered (takes a few minutes) using the command below before trying out the feature. 
```PowerShell
 Get-AzProviderFeature -FeatureName "EncryptionAtHost" -ProviderNamespace "Microsoft.Compute"  
```
## Restrictions
1. The feature is available only in the USCentralEUAP region.
2. You cannot enable the feature if you have enabled [Azure Disks Encryption (guest-VM encryption using bitlocker/VM-Decrypt)](https://docs.microsoft.com/en-us/azure/security/fundamentals/azure-disk-encryption-vms-vmss) for your VMs/VMSSes and vice versa.
3. You have to deallocate your existing VMs to enable the encryption. 
4. You can enable the encryption for existing VMSS. However, only new VMs created after enabling the encryption is encrypted. 
5. Legacy VM Sizes are not supported. You can find the list of supported VM sizes by:

   a. Calling the [Resource Skus API](https://docs.microsoft.com/en-us/rest/api/compute/resourceskus/list) and checking that the   EncryptionAtHostSupported capability is set to True
   ```json
        {
         "resourceType": "virtualMachines",
         "name": "Standard_DS1_v2",
         "tier": "Standard",
         "size": "DS1_v2",
         "family": "standardDSv2Family",
         "locations": [
           "CentralUSEUAP"
         ],
         "capabilities": [
           {
             "name": "EncryptionAtHostSupported",
             "value": "True"
           }
         ]
       }
    ```   
   b. Calling the [Get-AzComputeResourceSku](https://docs.microsoft.com/en-us/powershell/module/az.compute/get-azcomputeresourcesku?view=azps-3.8.0) PowerShell cmdlet 
    ```powershell
    $vmSizes=Get-AzComputeResourceSku | where{$_.ResourceType -eq 'virtualMachines' -and $_.Locations.Contains('CentralUSEUAP')} 

   foreach($vmSize in $vmSizes)
   {
       foreach($capability in $vmSize.capabilities)
       {
           if($capability.Name -eq 'EncryptionAtHostSupported' -and $capability.Value -eq 'true')
           {
               $vmSize

           }

       }
   }
   ```
6. VM Size upgrade will result in validation to check if the new VM size supports the EncryptionAtHost feature.

## Enable end to end encryption for disks attached to a VM with customer managed keys (CMK)

1. Follow the instructions [here](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/disk-encryption#setting-up-your-azure-key-vault-and-diskencryptionset) for creating a Key Vault for storing your keys and a DiskEncryptionSet pointing to a key in the Key Vault

2. Create a VM with managed disks by passing the resource URI of the DiskEncryptionSet created in the step #1 to the the sample template [CreateVMWithDisksEncryptedInTransitAtRestWithCMK.json](https://github.com/ramankumarlive/manageddisksendtoendencryptionpreview/blob/master/CreateVMWithDisksEncryptedInTransitAtRestWithCMK.json)

 ```PowerShell
 $password=ConvertTo-SecureString -String "yourPassword" -AsPlainText -Force
 New-AzResourceGroupDeployment -ResourceGroupName yourResourceGroupName `
   -TemplateUri "https://raw.githubusercontent.com/ramankumarlive/manageddisksendtoendencryptionpreview/master/CreateVMWithDisksEncryptedInTransitAtRestWithCMK.json" `
   -virtualMachineName "yourVMName" `
   -adminPassword $password `
   -vmSize "Standard_DS3_V2" `
   -diskEncryptionSetId "/subscriptions/dd80b94e-0463-4a65-8d04-c94f403879dc/resourceGroups/yourResourceGroupName/providers/Microsoft.Compute/diskEncryptionSets/yourDESName" `
   -region "CentralUSEUAP"
 ```


## Enable end to end encryption for disks attached to a VM with platform managed keys (PMK)

1. Create a VM with managed disks using the sample template [CreateVMWithDisksEncryptedInTransitAtRestWithPMK.json](https://github.com/ramankumarlive/manageddisksendtoendencryptionpreview/blob/master/CreateVMWithDisksEncryptedInTransitAtRestWithPMK.json)

 ```PowerShell
 $password=ConvertTo-SecureString -String "Password@123" -AsPlainText -Force
 New-AzResourceGroupDeployment -ResourceGroupName CMKTesting `
   -TemplateUri "https://raw.githubusercontent.com/ramankumarlive/manageddisksendtoendencryptionpreview/master/CreateVMWithDisksEncryptedInTransitAtRestWithPMK.json" `
   -virtualMachineName "ramane2evm12" `
   -adminPassword $password `
   -vmSize "Standard_DS3_V2" `
   -region "CentralUSEUAP"
 ```
