---
layout: post
title: "Dumping All the Resources in Resource Group using Ansible"
date: 2019-03-22 15:25:37 +0800
comments: true
categories: 
---

Now, this is pretty cool. With a very small Ansible playbook you can dump entire content of a Resource Group.

Please note it requires Ansible 2.8.

## Running the Example

The example is now available in official **Azure-Samples**:

    https://github.com/Azure-Samples/ansible-playbooks/blob/master/rest/resourcegroup_dump_resources.yml

## The Playbook

The playbook uses **azure_rm_resource_facts** module and does following steps:

- retrieve list of all of the resources in the resource group
- dump list of resources
- create list of resource ids
- dump list of resource ids
- iterate through resource ids and retrieve details of each resource
- convert output to single list of all resources
- dump detailed list of resources

{% raw %}
```yaml
    - name: List all the resources under a resource group
      azure_rm_resource_facts:
        resource_group: zimsdtl
        resource_type: resources
      register: output

    - name: Dump raw list of resouces in the resource group
      debug:
        var: output

    - name: Create a list of resource IDs
      set_fact:
        resource_ids: "{{ output.response | map(attribute='id') | list }}"

    - name: Dump list of resource IDs
      debug:
        var: resource_ids

    - name: Iterate through the resources and query details on each one
      azure_rm_resource_facts:
        url: "{{ item }}"
      register: output
      with_items: "{{ resource_ids }}"

    - name: Create single list of resources
      set_fact:
        resources: "{{ output.results | map(attribute='response') | sum(start=[]) }}"

    - name: Dump final list of resources
      debug:
        var: resources
```
{% endraw %}

## Final Output

And final output looks like this:

{% raw %}
```
PLAY [List all resources in resource group] *********************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************
ok: [localhost]

TASK [List all the resources under a resource group] ************************************************************************************************************************************
 [WARNING]: Azure API profile latest does not define an entry for GenericRestClient

ok: [localhost]

TASK [Dump raw list of resouces in the resource group] **********************************************************************************************************************************
ok: [localhost] => {
    "output": {
        "changed": false,
        "failed": false,
        "response": [
            {
                "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.DevTestLab/labs/labxxxx",
                "location": "eastus",
                "name": "labxxxx",
                "type": "Microsoft.DevTestLab/labs"
            },
            {
                "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.DevTestLab/labs/labxxxx/virtualMachines/vmxxxx",
                "location": "eastus",
                "name": "labxxxx/vmxxxx",
                "type": "Microsoft.DevTestLab/labs/virtualMachines"
            },
            {
                "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.KeyVault/vaults/labxxxx9607",
                "location": "eastus",
                "name": "labxxxx9607",
                "tags": {
                    "CreatedBy": "DevTestLabs",
                    "hidden-DevTestLabs-LabUId": "1f33aff8-4e19-43e2-b2fc-181ce85afd2a"
                },
                "type": "Microsoft.KeyVault/vaults"
            },
            {
                "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.Network/virtualNetworks/vnxxxx",
                "location": "eastus",
                "name": "vnxxxx",
                "tags": {
                    "hidden-DevTestLabs-LabUId": "1f33aff8-4e19-43e2-b2fc-181ce85afd2a"
                },
                "type": "Microsoft.Network/virtualNetworks"
            },
            {
                "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.Storage/storageAccounts/alabxxxx2409",
                "kind": "StorageV2",
                "location": "eastus",
                "name": "alabxxxx2409",
                "sku": {
                    "name": "Standard_LRS",
                    "tier": "Standard"
                },
                "tags": {
                    "hidden-DevTestLabs-LabUId": "1f33aff8-4e19-43e2-b2fc-181ce85afd2a"
                },
                "type": "Microsoft.Storage/storageAccounts"
            }
        ],
        "url": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/resources",
        "warnings": [
            "Azure API profile latest does not define an entry for GenericRestClient"
        ]
    }
}

TASK [Create a list of resource IDs] ****************************************************************************************************************************************************
ok: [localhost]

TASK [Dump list of resource IDs] ********************************************************************************************************************************************************
ok: [localhost] => {
    "resource_ids": [
        "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.DevTestLab/labs/labxxxx",
        "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.DevTestLab/labs/labxxxx/virtualMachines/vmxxxx",
        "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.KeyVault/vaults/labxxxx9607",
        "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.Network/virtualNetworks/vnxxxx",
        "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.Storage/storageAccounts/alabxxxx2409"
    ]
}

TASK [Iterate through the resources and query details on each one] **********************************************************************************************************************
ok: [localhost] => (item=/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.DevTestLab/labs/labxxxx)
ok: [localhost] => (item=/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.DevTestLab/labs/labxxxx/virtualMachines/vmxxxx)
ok: [localhost] => (item=/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.KeyVault/vaults/labxxxx9607)
ok: [localhost] => (item=/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.Network/virtualNetworks/vnxxxx)
ok: [localhost] => (item=/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.Storage/storageAccounts/alabxxxx2409)

TASK [Create single list of resources] **************************************************************************************************************************************************
ok: [localhost]

TASK [Dump final list of resources] *****************************************************************************************************************************************************
ok: [localhost] => {
    "resources": [
        {
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/zimsdtl/providers/microsoft.devtestlab/labs/labxxxx",
            "location": "eastus",
            "name": "labxxxx",
            "properties": {
                "announcement": {
                    "enabled": "Disabled",
                    "expired": false,
                    "markdown": "",
                    "title": ""
                },
                "artifactsStorageAccount": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.Storage/storageAccounts/alabxxxx2409",
                "createdDate": "2019-03-01T08:42:35.4625319+00:00",
                "defaultPremiumStorageAccount": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.Storage/storageAccounts/alabxxxx2409",
                "defaultStorageAccount": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.Storage/storageAccounts/alabxxxx2409",
                "environmentPermission": "Reader",
                "labStorageType": "Standard",
                "mandatoryArtifactsResourceIdsLinux": [],
                "mandatoryArtifactsResourceIdsWindows": [],
                "premiumDataDiskStorageAccount": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.Storage/storageAccounts/alabxxxx2409",
                "premiumDataDisks": "Disabled",
                "provisioningState": "Succeeded",
                "support": {
                    "enabled": "Disabled",
                    "markdown": ""
                },
                "uniqueIdentifier": "1f33aff8-4e19-43e2-b2fc-181ce85afd2a",
                "vaultName": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.KeyVault/vaults/labxxxx9607"
            },
            "type": "Microsoft.DevTestLab/labs"
        },
        {
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/zimsdtl/providers/microsoft.devtestlab/labs/labxxxx/virtualmachines/vmxxxx",
            "location": "eastus",
            "name": "vmxxxx",
            "properties": {
                "allowClaim": false,
                "artifactDeploymentStatus": {
                    "artifactsApplied": 0,
                    "totalArtifacts": 0
                },
                "computeId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/labxxxx-vmxxxx-561112/providers/Microsoft.Compute/virtualMachines/vmxxxx",
                "createdByUser": "",
                "createdByUserId": "",
                "createdDate": "2019-03-01T08:44:56.9375632+00:00",
                "dataDiskParameters": [],
                "disallowPublicIpAddress": false,
                "expirationDate": "2029-02-22T01:49:12.117974+00:00",
                "fqdn": "vmxxxx.eastus.cloudapp.azure.com",
                "galleryImageReference": {
                    "offer": "UbuntuServer",
                    "osType": "Linux",
                    "publisher": "Canonical",
                    "sku": "16.04-LTS",
                    "version": "latest"
                },
                "labSubnetName": "vnxxxxSubnet",
                "labVirtualNetworkId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.DevTestLab/labs/labxxxx/virtualnetworks/vnxxxx",
                "lastKnownPowerState": "Stopped",
                "networkInterface": {},
                "notes": "Virtual machine notes, just something....",
                "osType": "Linux",
                "ownerObjectId": "20d81029-94cd-4923-a766-994415ff73bd",
                "ownerUserPrincipalName": "",
                "provisioningState": "Succeeded",
                "size": "Standard_A2_v2",
                "storageType": "Standard",
                "uniqueIdentifier": "39fc5a87-2b95-48ff-a827-468902b78149",
                "userName": "dtladmin",
                "virtualMachineCreationSource": "FromGalleryImage"
            },
            "type": "Microsoft.DevTestLab/labs/virtualMachines"
        },
        {
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.KeyVault/vaults/labxxxx9607",
            "location": "eastus",
            "name": "labxxxx9607",
            "properties": {
                "accessPolicies": [
                    {
                        "objectId": "9d706dfc-a44a-4341-9cac-e995cc8a1530",
                        "permissions": {
                            "secrets": [
                                "all"
                            ]
                        },
                        "tenantId": "72f988bf-86f1-41af-91ab-2d7cd011db47"
                    }
                ],
                "enabledForDeployment": true,
                "provisioningState": "Succeeded",
                "sku": {
                    "family": "A",
                    "name": "standard"
                },
                "tenantId": "72f988bf-86f1-41af-91ab-2d7cd011db47",
                "vaultUri": "https://labxxxx9607.vault.azure.net/"
            },
            "tags": {
                "CreatedBy": "DevTestLabs",
                "hidden-DevTestLabs-LabUId": "1f33aff8-4e19-43e2-b2fc-181ce85afd2a"
            },
            "type": "Microsoft.KeyVault/vaults"
        },
        {
            "etag": "W/\"806f86f0-0cdd-4424-9fbc-17b9ab331ad7\"",
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.Network/virtualNetworks/vnxxxx",
            "location": "eastus",
            "name": "vnxxxx",
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "10.0.0.0/20"
                    ]
                },
                "dhcpOptions": {
                    "dnsServers": []
                },
                "enableDdosProtection": false,
                "enableVmProtection": false,
                "provisioningState": "Succeeded",
                "resourceGuid": "9f622d47-478c-44a1-9796-3ba51eb44259",
                "subnets": [
                    {
                        "etag": "W/\"806f86f0-0cdd-4424-9fbc-17b9ab331ad7\"",
                        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.Network/virtualNetworks/vnxxxx/subnets/vnxxxxSubnet",
                        "name": "vnxxxxSubnet",
                        "properties": {
                            "addressPrefix": "10.0.0.0/20",
                            "delegations": [],
                            "provisioningState": "Succeeded"
                        },
                        "type": "Microsoft.Network/virtualNetworks/subnets"
                    }
                ],
                "virtualNetworkPeerings": []
            },
            "tags": {
                "hidden-DevTestLabs-LabUId": "1f33aff8-4e19-43e2-b2fc-181ce85afd2a"
            },
            "type": "Microsoft.Network/virtualNetworks"
        },
        {
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdtl/providers/Microsoft.Storage/storageAccounts/alabxxxx2409",
            "kind": "StorageV2",
            "location": "eastus",
            "name": "alabxxxx2409",
            "properties": {
                "accessTier": "Hot",
                "creationTime": "2019-03-01T08:43:18.0722775Z",
                "encryption": {
                    "keySource": "Microsoft.Storage",
                    "services": {
                        "blob": {
                            "enabled": true,
                            "lastEnabledTime": "2019-03-01T08:43:18.1972664Z"
                        },
                        "file": {
                            "enabled": true,
                            "lastEnabledTime": "2019-03-01T08:43:18.1972664Z"
                        }
                    }
                },
                "networkAcls": {
                    "bypass": "AzureServices",
                    "defaultAction": "Allow",
                    "ipRules": [],
                    "virtualNetworkRules": []
                },
                "primaryEndpoints": {
                    "blob": "https://alabxxxx2409.blob.core.windows.net/",
                    "dfs": "https://alabxxxx2409.dfs.core.windows.net/",
                    "file": "https://alabxxxx2409.file.core.windows.net/",
                    "queue": "https://alabxxxx2409.queue.core.windows.net/",
                    "table": "https://alabxxxx2409.table.core.windows.net/",
                    "web": "https://alabxxxx2409.z13.web.core.windows.net/"
                },
                "primaryLocation": "eastus",
                "provisioningState": "Succeeded",
                "statusOfPrimary": "available",
                "supportsHttpsTrafficOnly": true
            },
            "sku": {
                "name": "Standard_LRS",
                "tier": "Standard"
            },
            "tags": {
                "hidden-DevTestLabs-LabUId": "1f33aff8-4e19-43e2-b2fc-181ce85afd2a"
            },
            "type": "Microsoft.Storage/storageAccounts"
        }
    ]
}

PLAY RECAP ******************************************************************************************************************************************************************************
localhost                  : ok=8    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
{% endraw %}
