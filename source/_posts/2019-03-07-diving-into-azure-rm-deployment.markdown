---
layout: post
title: "Diving into azure_rm_deployment"
date: 2019-03-07 10:40:24 +0800
comments: true
categories: 
---

It's time to refresh **azure_rm_deployment**.

Currently we have following problems with **azure_rm_deployment**

- it's a kind of VM oriented only
- no proper facts module, no proper information about created resources, so it's difficult to build on the top of resources created by ARM deployment
- deleting deployment actually deletes entire resource group, so no convenient way to have any additional resources in the same resource group
- default deployment name (user can optionally specify) is confusing and inconsistent with other resources

What we can fix, and it's quite easy:
- facts module that lists all the resource ids in dictionary by resource type
- properly deleting resources created by deployment, enabling users to have more than one deployment in a resource group

Let's start with a simple playbook. I have just taken it from integration tests:

## How it currently works?

Just a simple playbook that creates a virtual machine:

```yaml
- name: Create Azure Deploy
    azure_rm_deployment:
    resource_group: "{{ resource_group }}"
    location: eastus2
    template_link: 'https://raw.githubusercontent.com/Azure/azure-quickstart-templates/d01a5c06f4f1bc03a049ca17bbbd6e06d62657b3/101-vm-simple-linux/azuredeploy.json'
    deployment_name: "{{ dns_label }}"
    parameters:
        adminUsername:
        value: chouseknecht
        adminPassword:
        value: password123!
        dnsLabelPrefix:
        value: "{{ dns_label }}"
        ubuntuOSVersion:
        value: "16.04.0-LTS"
    register: output

- debug:
    var: output
```

and the output looks like this:

```json
{
    "output": {
        "changed": true,
        "deployment": {
            "group_name": "zimsdeployment",
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdeployment/providers/Microsoft.Resources/deployments/zimsdeployment",
            "instances": [
                {
                    "ips": [
                        {
                            "dns_settings": {
                                "domain_name_label": "zimsdeployment",
                                "fqdn": "zimsdeployment.eastus2.cloudapp.azure.com"
                            },
                            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsdeployment/providers/Microsoft.Network/publicIPAddresses/myPublicIP",
                            "name": "myPublicIP",
                            "public_ip": "23.100.65.195",
                            "public_ip_allocation_method": "Dynamic"
                        }
                    ],
                    "vm_name": "MyUbuntuVM"
                }
            ],
            "name": "zimsdeployment",
            "outputs": {
                "hostname": {
                    "type": "String",
                    "value": "zimsdeployment.eastus2.cloudapp.azure.com"
                },
                "sshCommand": {
                    "type": "String",
                    "value": "ssh chouseknecht@zimsdeployment.eastus2.cloudapp.azure.com"
                }
            }
        },
        "failed": false,
        "msg": "deployment succeeded"
    }
}
```

## What is actually returned by Azure REST API?

I have created following task to find it out:

```yaml
- name: Get deployment
    azure_rm_resource_facts:
    api_version: '2019-03-01'
    resource_group: "{{ resource_group }}"
    provider: resources
    resource_type: deployments
    resource_name:  "{{ dns_label }}"
    register: output
- debug:
    var: output
```

As you see below it actually returns quite a lot:
- list of all created resource IDs
- list of all dependencies between resources

```json
{
    "output": {
        "changed": false,
        "failed": false,
        "response": [
            {
                "id": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimsanotherdeployment/providers/Microsoft.Resources/deployments/zimsdplxxxx",
                "name": "zimsdplxxxx",
                "properties": {
                    "correlationId": "0c26c003-f147-4f18-95c6-a28e43630dd1",
                    "dependencies": [
                        {
                            "dependsOn": [
                                {
                                    "id": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimsanotherdeployment/providers/Microsoft.Network/publicIPAddresses/myPublicIP",
                                    "resourceName": "myPublicIP",
                                    "resourceType": "Microsoft.Network/publicIPAddresses"
                                },
                                {
                                    "id": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimsanotherdeployment/providers/Microsoft.Network/virtualNetworks/MyVNET",
                                    "resourceName": "MyVNET",
                                    "resourceType": "Microsoft.Network/virtualNetworks"
                                }
                            ],
                            "id": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimsanotherdeployment/providers/Microsoft.Network/networkInterfaces/myVMNic",
                            "resourceName": "myVMNic",
                            "resourceType": "Microsoft.Network/networkInterfaces"
                        },
                        {
                            "dependsOn": [
                                {
                                    "id": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimsanotherdeployment/providers/Microsoft.Storage/storageAccounts/iovbjgl443l64sawinvm",
                                    "resourceName": "iovbjgl443l64sawinvm",
                                    "resourceType": "Microsoft.Storage/storageAccounts"
                                },
                                {
                                    "id": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimsanotherdeployment/providers/Microsoft.Network/networkInterfaces/myVMNic",
                                    "resourceName": "myVMNic",
                                    "resourceType": "Microsoft.Network/networkInterfaces"
                                }
                            ],
                            "id": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimsanotherdeployment/providers/Microsoft.Compute/virtualMachines/SimpleWinVM",
                            "resourceName": "SimpleWinVM",
                            "resourceType": "Microsoft.Compute/virtualMachines"
                        }
                    ],
                    "duration": "PT46.927468S",
                    "mode": "Incremental",
                    "outputResources": [
                        {
                            "id": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimsanotherdeployment/providers/Microsoft.Compute/virtualMachines/SimpleWinVM"
                        },
                        {
                            "id": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimsanotherdeployment/providers/Microsoft.Network/networkInterfaces/myVMNic"
                        },
                        {
                            "id": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimsanotherdeployment/providers/Microsoft.Network/publicIPAddresses/myPublicIP"
                        },
                        {
                            "id": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimsanotherdeployment/providers/Microsoft.Network/virtualNetworks/MyVNET"
                        },
                        {
                            "id": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimsanotherdeployment/providers/Microsoft.Storage/storageAccounts/iovbjgl443l64sawinvm"
                        }
                    ],
                    "outputs": {
                        "hostname": {
                            "type": "String",
                            "value": "zimsdplxxxx.eastus2.cloudapp.azure.com"
                        }
                    },
                    "parameters": {
                        "adminPassword": {
                            "type": "SecureString"
                        },
                        "adminUsername": {
                            "type": "String",
                            "value": "chouseknecht"
                        },
                        "dnsLabelPrefix": {
                            "type": "String",
                            "value": "zimsdplxxxx"
                        },
                        "location": {
                            "type": "String",
                            "value": "eastus2"
                        },
                        "windowsOSVersion": {
                            "type": "String",
                            "value": "2016-Datacenter"
                        }
                    },
                    "providers": [
                        {
                            "namespace": "Microsoft.Storage",
                            "resourceTypes": [
                                {
                                    "locations": [
                                        "eastus2"
                                    ],
                                    "resourceType": "storageAccounts"
                                }
                            ]
                        },
                        {
                            "namespace": "Microsoft.Network",
                            "resourceTypes": [
                                {
                                    "locations": [
                                        "eastus2"
                                    ],
                                    "resourceType": "publicIPAddresses"
                                },
                                {
                                    "locations": [
                                        "eastus2"
                                    ],
                                    "resourceType": "virtualNetworks"
                                },
                                {
                                    "locations": [
                                        "eastus2"
                                    ],
                                    "resourceType": "networkInterfaces"
                                }
                            ]
                        },
                        {
                            "namespace": "Microsoft.Compute",
                            "resourceTypes": [
                                {
                                    "locations": [
                                        "eastus2"
                                    ],
                                    "resourceType": "virtualMachines"
                                }
                            ]
                        }
                    ],
                    "provisioningState": "Succeeded",
                    "templateHash": "17224794003184781395",
                    "templateLink": {
                        "contentVersion": "1.0.0.0",
                        "uri": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json"
                    },
                    "timestamp": "2019-03-07T05:49:05.3868647Z"
                },
                "type": "Microsoft.Resources/deployments"
            }
        ],
        "url": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimsanotherdeployment/providers/Microsoft.resources/deployments/zimsdplxxxx",
        "warnings": [
            "Azure API profile latest does not define an entry for GenericRestClient"
        ]
    }
}
```
