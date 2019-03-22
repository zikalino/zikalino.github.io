---
layout: post
title: "New Ansible module azure_rm_hdinsightcluster"
date: 2019-03-22 09:00:35 +0800
comments: true
categories: 
---

https://github.com/Azure-Samples/ansible-playbooks/blob/master/hdinsight-create.yml


This sample will:

- create resource group
- create storage account
- obtain storage account keys required by the cluster
- create HDInsight cluster
- resize cluster
- delete cluster

## Create Random Prefix

{% raw %}
```yaml
- hosts: localhost
  tasks:
    - name: Prepare random prefix
      set_fact:
        rpfx: "{{ resource_group_name | hash('md5') | truncate(7, True, '') }}{{ 1000 | random }}"
      run_once: yes
```
{% endraw %}

## Variables

{% raw %}
```yaml
  vars:
    resource_group: "{{ resource_group_name }}"
    location: eastus
    vnet_name: myVirtualNetwork
    subnet_name: mySubnet
    cluster_name: mycluster{{ rpfx }}
    storage_account_name: mystorage{{ rpfx }}
```
{% endraw %}

## Create Resource Group

{% raw %}
```yaml
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"
```
{% endraw %}

## Create Storage Account and Obtain Keys

{% raw %}
```yaml
    - name: Create storage account
      azure_rm_storageaccount:
          resource_group: "{{ resource_group }}"
          name: "{{ storage_account_name }}"
          account_type: Standard_LRS
          location: eastus2
```
{% endraw %}

{% raw %}
```yaml
    - name: Get storage account keys
      azure_rm_resource:
        api_version: '2018-07-01'
        method: POST
        resource_group: "{{ resource_group }}"
        provider: storage
        resource_type: storageaccounts
        resource_name: "{{ storage_account_name }}"
        subresource:
          - type: listkeys
      register: storage_output

    - debug:
        var: storage_output
```
{% endraw %}

## Create Instance of HDInsight Cluster

{% raw %}
```yaml
    - name: Create instance of Cluster
      azure_rm_hdinsightcluster:
        resource_group: "{{ resource_group }}"
        name: "{{ cluster_name }}"
        location: eastus2
        cluster_version: 3.6
        os_type: linux
        tier: standard
        cluster_definition:
          kind: spark
          gateway_rest_username: http-user
          gateway_rest_password: MuABCPassword!!@123 
        storage_accounts:
          - name: "{{ storage_account_name }}.blob.core.windows.net" 
            is_default: yes
            container: "{{ cluster_name }}"
            key: "{{ storage_output['response']['keys'][0]['value'] }}"
        compute_profile_roles:
          - name: headnode
            target_instance_count: 1
            vm_size: Standard_D3
            linux_profile:
              username: sshuser
              password: MuABCPassword!!@123
          - name: workernode
            target_instance_count: 1
            vm_size: Standard_D3
            linux_profile:
              username: sshuser
              password: MuABCPassword!!@123
          - name: zookeepernode
            target_instance_count: 3
            vm_size: Medium
            linux_profile:
              username: sshuser
              password: MuABCPassword!!@123
```
{% endraw %}

## Resize Cluster

The only thing that can be changed after HDInsight cluster is created is the number of worker nodes.
Below task will increment number of nodes from 1 to 2.

{% raw %}
```yaml
    - name: Resize cluster
      azure_rm_hdinsightcluster:
        resource_group: "{{ resource_group }}"
        name: "{{ cluster_name }}"
        location: eastus2
        cluster_version: 3.6
        os_type: linux
        tier: standard
        cluster_definition:
          kind: spark
          gateway_rest_username: http-user
          gateway_rest_password: MuABCPassword!!@123 
        storage_accounts:
          - name: "{{ storage_account_name }}.blob.core.windows.net"
            is_default: yes
            container: "{{ cluster_name }}"
            key: "{{ storage_output['response']['keys'][0]['value'] }}"
        compute_profile_roles:
          - name: headnode
            target_instance_count: 1
            vm_size: Standard_D3
            linux_profile:
              username: sshuser
              password: MuABCPassword!!@123
          - name: workernode
            target_instance_count: 2
            vm_size: Standard_D3
            linux_profile:
              username: sshuser
              password: MuABCPassword!!@123
          - name: zookeepernode
            target_instance_count: 3
            vm_size: Medium
            linux_profile:
              username: sshuser
              password: MuABCPassword!!@123
        tags:
          aaa: bbb
      register: output
```
{% endraw %}

## Clean Up

{% raw %}
```yaml
    - name: Delete instance of Cluster
      azure_rm_hdinsightcluster:
        resource_group: "{{ resource_group }}"
        name: "{{ cluster_name }}"
        state: absent
```
{% endraw %}
