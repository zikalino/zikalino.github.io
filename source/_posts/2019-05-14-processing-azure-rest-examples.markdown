---
layout: post
title: "Processing Azure REST Examples"
date: 2019-05-14 11:26:13 -0700
comments: true
categories: 
---

## Some Issues with Azure REST API Examples

- Examples are currently not tested
- No naming conventions
- Missing Examples (for instance Web App, Function App, I have created missing Resource Group example myself)
- No dependencies

## Solutions

### Example naming conventions

I am using resource URIs to create sample names. For instance:

```
network_virtualnetworks_subnets_put.yml
network_virtualnetworks_put.yml
resourcegroups_put.yml
```

This approach makes it easy to find dependencies between examples, which is shown in the demo below.

### Resource naming conventions within samples

I am overriding names of the resources with names derived from URI pattern.

For instance resource group will be always called **myresourcegroup**, and this makes combining examples trivial.

### Extend Example Validation

More checks can be introduced to validate examples against actual REST API definition to detect mistakes. I will include more examples here.

### Encourage Reusing REST API Examples Downstream

Currently examples from REST API are not used directly in any other project. Encouraging usage, testing and contributing fixes upstream would significantly improve REST API specs for everyone.

## What we do in Ansible?

We are currently developing Autorest / Magic Modules solution to generate Ansible and Terraform modules.

Ansible modules should come with appropriate examples, and also having integration tests are requirement.

Before:

- examples were usually written manually
- examples were sometimes broken or getting outdated
- integration tests were written manually

Where we are going to do now:

- we want to have requirement that examples included in the module are **all** included/kept in sync with integration test
- examples and integration tests are **not** written manually

Benefits:

- Azure REST API Examples are tested
- Examples are fixed/added at the source which is beneficial to everyone
- Ensuring that Azure REST API documentation has actually complete/correct examples
- Ansible/Terraform documentation has complete and correct examples that are inline with generic Azure documentation

## What could be done next?

- integrate autorest.devops autorest extension with Azure REST API Specs CI
- enforce new rules / possibly generate warnings when necessary?
- calculate API score for every resource including example coverage, following naming conventions, etc.

## Sample Processed Output

### Subnet Example

Below you can see **network_virtualnetworks_subnet_put.yml** example.

On the top you can see that it automatically includes **network_virtualnetworks_put.yml** example that creates virtual network.

{% raw %}
```
- import_playbook: network_virtualnetworks_put.yml
- hosts: localhost
  roles:
    - ../modules
  vars_files:
    - _vars.yml
  vars:
    resource_group: myresourcegroup
    virtual_network_name: myvirtualnetwork
    subnet_name: mysubnet
  tasks:
    - name: Create subnet
      azure_rm_resource:
        idempotency: yes
        api_version: '2018-02-01'
        polling_timeout: 600
        polling_interval: 30
        # url: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/virtualNetworks/{virtualNetworkName}/subnets/{subnetName}
        resource_group: '{{ resource_group }}'
        provider: Network
        resource_type: virtualNetworks
        resource_name: "{{ virtual_network_name }}"
        subresource:
          - type: subnets
            name: "{{ subnet_name }}"
        body:
          properties:
            addressPrefix: "10.0.0.0/16"
```
{% endraw %}

### Virtual Network Example

Below you can see **network_virtualnetwork_put.yml** example that was referred in previous playbook.

On the top you can see that it automatically includes **resourcegroups_put.yml** example that creates required .

{% raw %}
```
- import_playbook: resourcegroups_put.yml
- hosts: localhost
  roles:
    - ../modules
  vars_files:
    - _vars.yml
  vars:
    resource_group: myresourcegroup
    virtual_network_name: myvirtualnetwork
  tasks:
    - name: Create virtual network
      azure_rm_resource:
        idempotency: yes
        api_version: '2018-02-01'
        polling_timeout: 600
        polling_interval: 30
        # url: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/virtualNetworks/{virtualNetworkName}
        resource_group: '{{ resource_group }}'
        provider: Network
        resource_type: virtualNetworks
        resource_name: "{{ virtual_network_name }}"
        body:
          properties:
            addressSpace:
              addressPrefixes:
                - "10.0.0.0/16"
          location: "eastus"
```
{% endraw %}

### Resource Group Example

And finally you can check **resource_groups_put.yml** example that is required by previous playbooks.

On the top you can see that it automatically includes **resourcegroups_put.yml** example that creates required.

You can find a PR here that I have merged to REST API specs:

https://github.com/Azure/azure-rest-api-specs/pull/4971

You can find the same example was propagated here to Azure REST API docs:

https://docs.microsoft.com/en-us/rest/api/resources/resourcegroups/createorupdate

{% raw %}
```
- hosts: localhost
  roles:
    - ../modules
  vars_files:
    - _vars.yml
  vars:
    resource_group: myresourcegroup
  tasks:
    - name: Create or update a resource group.
      azure_rm_resource:
        idempotency: yes
        api_version: '2018-05-01'
        # url: /subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}
        resource_group: '{{ resource_group }}'
        body:
          location: eastus
```
{% endraw %}

## Demo

### Prerequisites

Ansible 2.8 

{% raw %}
```
pip install ansible[azure]
```
{% endraw %}

Clone sample repository:

{% raw %}
```
git clone https://github.com/zikalino/ansible-azure-rest-examples.git
cd examples
```
{% endraw %}

### Running Examples Demo

From examples folder run:

{% raw %}
```
ansible-playbook network_virtual_networks_subnet_put.yml
```
{% endraw %}

You should see that all dependencies and subnet are correctly created:

![Running Examples Demo](/images/running-examples.png)
