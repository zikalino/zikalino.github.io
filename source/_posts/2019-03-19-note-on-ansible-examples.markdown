---
layout: post
title: "Note on Ansible Examples"
date: 2019-03-19 06:42:26 +0800
comments: true
categories: 
---

When I started working with Ansible I wa

Let's do following exercise. I want to create an Azure Virtual Machine using examples from documentation.

Let's start with default virtual machine example (https://docs.ansible.com/ansible/latest/modules/azure_rm_virtualmachine_module.html)

```yaml
- name: Create Azure VM using default samples
  hosts: localhost
  connection: local
  tasks:
    - name: Create VM with defaults
      azure_rm_virtualmachine:
        resource_group: Testing
        name: testvm10
        admin_username: chouseknecht
        admin_password: <your password here>
        image:
          offer: CentOS
          publisher: OpenLogic
          sku: '7.1'
          version: latest
```

Now, Ansible will complain that resource group **Testing** is not present, so I add resource group from here https://docs.ansible.com/ansible/latest/modules/azure_rm_resourcegroup_module.html:

```yaml
- name: Create Azure VM using default samples
  hosts: localhost
  connection: local
  tasks:
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: Testing
        location: westus
        tags:
          testing: testing
          delete: never
    - name: Create VM with defaults
      azure_rm_virtualmachine:
        resource_group: Testing
        name: testvm10
        admin_username: chouseknecht
        admin_password: <your password here>
        image:
          offer: CentOS
          publisher: OpenLogic
          sku: '7.1'
          version: latest
```
