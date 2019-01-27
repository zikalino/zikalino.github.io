---
layout: post
title: "Installing HTTPD on VMSS using VMSS Extension"
date: 2019-01-24 09:33:49 +0800
comments: true
categories: 
---

There are two (or three) proper ways of updating Virtual Machine Scale Set instances:

- use custom images and update instances whenever something changes
- use VMSS Extension to install additional software on all the VM instances
- combine both

Possibly there's a fourth way, but I haven't investigated that yet. I am thinking of updating VMSS data disks on the fly, but I haven't tried that yet. Let's focus on using VMSS Extension. In this article I will show how to use new **azure_rm_virtualmachinescalesetextension** module. It's already available in **azure_preview_modules** and will be released in **Ansible 2.8**.

## Prerequisites

Before trying the playbooks, clone following repository:

```bash
git clone https://github.com/Azure-Samples/ansible-playbooks.git
cd ansible-playbooks/vmss_extension
```


## Step 1: Create VMSS using Ubuntu 16.04 Image

In this step we will just create Azure Virtual Machine Scaleset based using standard Ubuntu 16.04 image and all prerequisites.

Run following playbook:

```
ansible-playbook 01-create-vmss.yml --extra-vars "resource_group=myrg"
```

After this playbook is executed you will see an IP addres. Copy it to the browser. The page should fail to open as initial Ubuntu 16.04 image doesn't have web server installed.

You can also go to Azure Portal and check the status of VMSS instances:

![VMSS Instances 1](/images/vmss-extension-01.png)

## Step 2: Install VMSS extension

In second step we will install a **Custom Script** VMSS extension that will install **httpd**.

Run following playbook:

```
ansible-playbook 02-setup-httpd.yml --extra-vars "resource_group=myrg"
```

In this step we use **azure_rm_virtualmachinescalesetextension** module:

```yaml
- name: Create VM Extension - HTTPD
  azure_rm_virtualmachinescalesetextension:
    resource_group: "{{ resource_group }}"
    vmss_name: "{{ vmss_name }}"
    name: testVMExtension
    publisher: Microsoft.Azure.Extensions
    type: CustomScript
    type_handler_version: 2.0
    auto_upgrade_minor_version: true
    settings: {"commandToExecute": "sudo apt-get -y install apache2"}
```

Please note that creating extension didn't update existing instances. You can check how it looks in the portal again - as you see all the instances are not up to date:

![VMSS Instances 2](/images/vmss-extension-02.png)

## Step 3: Update all the instances

In this step we will actually upgrade all VM instances. Run following script:

```
ansible-playbook 03-update-instances.yml --extra-vars "resource_group=myrg"
```

First **azure_rm_virtualmachinescalesetinstance_facts** module is used to get list of instances:

```yaml
- name: List all of the instances
  azure_rm_virtualmachinescalesetinstance_facts:
    resource_group: "{{ resource_group }}"
    vmss_name: "{{ vmss_name }}"
  register: instances
```
And then **azure_rm_virtualmachinescalesetinstance** module to set **latest_version** to **yes**:

```yaml
- name: manually upgrade all the instances 
  azure_rm_virtualmachinescalesetinstance:
    resource_group: "{{ resource_group }}"
    vmss_name: "{{ vmss_name }}"
    instance_id: "{{ item.instance_id }}"
    latest_model: yes
  with_items: "{{ instances.instances }}"
```

Again, you can see in the portal how instances are being updated one by one:

![VMSS Instances 3](/images/vmss-extension-03.png)

When update is completed reload the page, and you should see that this time default Apache index page is displayed:

![VMSS Instances 4](/images/vmss-extension-04.png)
