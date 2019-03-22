---
layout: post
title: "Removing resource locks using Ansible and Azure REST API"
date: 2019-03-18 14:51:20 +0800
comments: true
categories: 
---

This is one of the topic I had to look into. Removing locks appeared to be an issue while performing automatic clean up of the resources.

Currently there's no support for locks in Ansible, but I have tried to use **azure_rm_resource_facts** module to list locks on both resource group and subscription level, and then delete locks using **azure_rm_resource**.

To list all the locks in the resource group:

```yaml
  - name: List all the locks in the resource group
    azure_rm_resource_facts:
      api_version: '2016-09-01'
      resource_group: "{{ resource_group }}"
      provider: authorization
      resource_type: locks
    register: output
  
  - debug:
      var: output
```

or all resources in the subscription, just remove **resource_group** parameter:

```yaml
  - name: List all the locks in the resource group
    azure_rm_resource_facts:
      api_version: '2016-09-01'
      provider: authorization
      resource_type: locks
    register: output
  
  - debug:
      var: output
```

Output will look as follows:

```json
{
    "output": {
        "changed": false,
        "failed": false,
        "response": [
            {
                "id": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimslockedrg/providers/Microsoft.Network/virtualNetworks/zimslockedvb/providers/Microsoft.Authorization/locks/justalock",
                "name": "justalock",
                "properties": {
                    "level": "CanNotDelete",
                    "notes": "blabla"
                },
                "type": "Microsoft.Authorization/locks"
            }
        ],
        "url": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimslockedrg/providers/Microsoft.authorization/locks",
        "warnings": [
            "Azure API profile latest does not define an entry for GenericRestClient"
        ]
    }
}
```

Based on this output you can just delete all the locks one by one using **azure_rm_resource** module:

```yaml
  - name: Delete locks one by one
    azure_rm_resource:
      api_version: '2016-09-01'
      url: "{{ item.id }}"
      provider: authorization
      resource_type: locks
      state: absent
    with_items: "{{ output.response }}"
```

Please note that there's a bug in the versions of Ansible earlier than 2.8, and the last playbook needs small modification:

```yaml
  - name: Delete locks one by one
    azure_rm_resource:
      api_version: '2016-09-01'
      url: "{{ item.id }}"
      provider: authorization
      resource_type: locks
      state: absent
    with_items: "{{ output.response[0].value }}"
```

