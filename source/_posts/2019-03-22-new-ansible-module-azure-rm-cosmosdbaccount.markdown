---
layout: post
title: "New Ansible module azure_rm_cosmosdbaccount"
date: 2019-03-22 09:00:13 +0800
comments: true
categories: 
---

Cosmos DB Account sample is available here:

https://github.com/Azure-Samples/ansible-playbooks/blob/master/cosmosdb-create.yml

This sample:

- creates random postfix to use in CosmosDB account name
- creates resource group
- creates virtual network
- creates subnet
- creates CosmosDB account
- queries and prints CosmosDB keys

## Create Random Postfix

{% raw %}
```yaml
- hosts: localhost
  tasks:
    - name: Prepare random postfix
      set_fact:
        rpfx: "{{ 1000 | random }}"
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
    cosmosdbaccount_name: cosmos{{ rpfx }}
```
{% endraw %}

Following variables can be set in **vars** section of the playbook:

|Variable Name|Description|Notes|
|-------------|-----------|-----|
|resource_group|resource group where resources will be created|by default it's using **resource_group_name** parameter passed when running the playbook|
|location|location where resources should be created||
|vnet_name|virtual network name||
|subnet|subnet name||
|cosmosdbaccount_name|CosmosDB account name|should include only lowercase characters and be globally unique|

## Create Resource Group

This simple task creates a resource group if doesn't exist yet.

{% raw %}
```yaml
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"
```
{% endraw %}

## Create Virtual Network and Subnet

This task created prerequisites for CosmosDB account:

- virtual network
- subnet

{% raw %}
```yaml
    - name: Create virtual network
      azure_rm_virtualnetwork:
        resource_group: "{{ resource_group }}"
        name: "{{ vnet_name }}"
        address_prefixes_cidr:
          - 10.1.0.0/16
          - 172.100.0.0/16
        dns_servers:
          - 127.0.0.1
          - 127.0.0.3

    - name: Add subnet
      azure_rm_subnet:
        name: "{{ subnet_name }}"
        virtual_network_name: "{{ vnet_name }}"
        resource_group: "{{ resource_group }}"
        address_prefix_cidr: "10.1.0.0/24"
```
{% endraw %}

## Create CosmosDB Account

This task creates actual CosmosDB account.

{% raw %}
```yaml
    - name: Create instance of Cosmos DB Account
      azure_rm_cosmosdbaccount:
        resource_group: "{{ resource_group }}"
        name: "{{ cosmosdbaccount_name }}"
        location: "{{ location }}"
        kind: global_document_db
        geo_rep_locations:
          - name: eastus
            failover_priority: 0
          - name: westus
            failover_priority: 1
        database_account_offer_type: Standard
        is_virtual_network_filter_enabled: yes
        virtual_network_rules:
          - subnet:
              resource_group: "{{ resource_group }}"
              virtual_network_name: "{{ vnet_name }}"
              subnet_name: "{{ subnet_name }}"
            ignore_missing_vnet_service_endpoint: yes
        enable_automatic_failover: yes
```
{% endraw %}

## Retrieve Keys

This task shows how to retrieve CosmosDB account keys that should be used later by any application using the database:

{% raw %}
```yaml
    - name: Get Cosmos DB Account facts with keys
      azure_rm_cosmosdbaccount_facts:
        resource_group: "{{ resource_group }}"
        name: "{{ cosmosdbaccount_name }}"
        retrieve_keys: all
      register: output

    - name: Display Cosmos DB Acccount facts output
      debug:
        var: output
```
{% endraw %}

## Use in your application

...
