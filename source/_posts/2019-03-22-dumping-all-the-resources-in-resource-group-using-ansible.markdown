---
layout: post
title: "Dumping All the Resources in Resource Group using Ansible"
date: 2019-03-22 15:25:37 +0800
comments: true
categories: 
---

Now, this is pretty cool. With a very small Ansible playbook you can dump entire content of a Resource Group.

{% raw %}
```yaml
- name: List all resources in resource group
  hosts: localhost
  connection: local
  tasks:
    - name: List all the resources under a resource group
      azure_rm_resource_facts:
        api_version: '2015-01-01'
        resource_group: zimsdtl
        resource_type: resources
      register: output

    - debug:
        var: output

    - name: Iterate through the resources and query details on each one
      azure_rm_resource_facts:
        url: "{{ item.id }}"
      register: output
      with_items: "{{ output.response }}"

    - debug:
        var: output
```
{% endraw %}

First task gets a list of resource ids, which looks as follows:

{% raw %}
```
TASK [debug] ****************************************************************************************************************************************************************************
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
            }
.........................
```
{% endraw %}

Second task iterates through the list and does generic query of every resource to get all the details of every resource:

{% raw %}
```
TASK [debug] ****************************************************************************************************************************************************************************
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
```
{% endraw %}
