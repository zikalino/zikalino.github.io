---
layout: post
title: "Installing Ansible using cloud-init"
date: 2019-02-01 11:25:16 +0800
comments: true
categories: 
---

Recently was playing with **cloud-init** a bit, and it seems like a great and consistent way of setting up Ansible VMs.
Here's an example of creating a virtual machine via Azure portal and installing Ansible at once.

What you need to do is to go to **Guest Config** tag and add appropriate **cloud-config** playbook:

![Guest Config](/images/cloud-init-ansible.png)

In this example I am using following code:

```yaml
#cloud-config
packages:
  - python-pip
runcmd:
  - sudo pip install ansible
  - sudo ansible-galaxy install azure.azure_preview_modules
  - sudo pip install -r ~/.ansible/roles/azure.azure_preview_modules/files/requirements-azure.txt
```

It can be even shorted if you don't need Azure preview modules:

```yaml
#cloud-config
packages:
  - python-pip
runcmd:
  - sudo pip install ansible[azure]
```
