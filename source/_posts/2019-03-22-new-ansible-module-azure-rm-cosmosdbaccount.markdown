---
layout: post
title: "New Ansible module azure_rm_cosmosdbaccount"
date: 2019-03-22 09:00:13 +0800
comments: true
categories: 
---

This sample:
- creates random postfix to use in CosmosDB account name
- creates resource group
- creates virtual network
- creates subnet
- creates CosmosDB account
- queries and prints CosmosDB keys

## Create Random Postfix

```yaml
- hosts: localhost
  tasks:
    - name: Prepare random postfix
      set_fact:
        rpfx: "{{ 1000 | random }}"
      run_once: yes
```

## Variables

```yaml
  vars:
    resource_group: "{{ resource_group_name }}"
    location: eastus
    vnet_name: myVirtualNetwork
    subnet_name: mySubnet
    cosmosdbaccount_name: cosmos{{ rpfx }}
```

## Create Resource Group

```yaml
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"
```

## Create Virtual Network and Subnet

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

## Create CosmosDB Account

```yaml
    - name: Create instance of Cosmos DB Account
      azure_rm_cosmosdbaccount:
        resource_group: "{{ resource_group }}"
        name: "{{ cosmosdbaccount_name }}"
        location: eastus
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

## Retrieve Keys

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

## Use in your application

...
