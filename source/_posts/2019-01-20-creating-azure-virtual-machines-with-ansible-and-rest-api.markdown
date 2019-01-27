---
layout: post
title: "Creating Azure Virtual Machines with Ansible and REST API"
date: 2019-01-20 18:09:56 +0800
comments: true
categories: 
---
Just go to **examples** folder and run following playbook:

```bash
ansible-playbook compute_virtualmachines_put.yml
```

Please note that it will create all the necessary dependencies:

- resource group
- network interface
- public IP address
- virtual network with subnet

```yaml
- import_playbook: network_networkinterfaces_put.yml
- hosts: localhost
  roles:
    - ../modules
  vars:
    subscription_id: "{{ lookup('env', 'AZURE_SUBSCRIPTION_ID') }}"
    resource_group: myResourceGroup
    virtual_machine_name: myVirtualMachine
    network_interface_name: myNetworkInterface
  tasks:
    - name: Create a vm with password authentication.
      azure_rm_resource:
        idempotency: yes
        api_version: '2018-10-01'
        polling_timeout: 600
        polling_interval: 30
        # url: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}
        resource_group: '{{ resource_group }}'
        provider: Compute
        resource_type: virtualMachines
        resource_name: "{{ virtual_machine_name }}"
        body:
          location: eastus
          properties:
            hardwareProfile:
              vmSize: Standard_D3_v2
            storageProfile:
              imageReference:
                sku: 2016-Datacenter
                publisher: MicrosoftWindowsServer
                version: latest
                offer: WindowsServer
              osDisk:
                caching: ReadWrite
                managedDisk:
                  storageAccountType: Standard_LRS
                name: myVMosdisk
                createOption: FromImage
            osProfile:
              adminUsername: adminxyz
              computerName: myVM
              adminPassword: MooMoo123!!!!
            networkProfile:
              networkInterfaces:
                - id: "/subscriptions/{{ subscription_id }}/resourceGroups/{{ resource_group }}/providers/Microsoft.Network/networkInterfaces/{{ network_interface_name }}"
                  properties:
                    primary: True
```
