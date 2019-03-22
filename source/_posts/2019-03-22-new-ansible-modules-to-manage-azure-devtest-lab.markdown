---
layout: post
title: "New Ansible modules to manage Azure DevTest Lab"
date: 2019-03-22 09:01:02 +0800
comments: true
categories: 
---

The new sample containing tasks to create and manage Azure DevTest Lab is available here:

https://github.com/Azure-Samples/ansible-playbooks/blob/master/devtestlab-create.yml

This sample will:

- create resource group
- create DevTest Lab
- create sample DevTest Lab policy
- create sample DevTest Lab schedule 
- create sample DevTest Lab Virtual Network
- create sample DevTest Lab artifact source
- create instance of DTL Virtual Machine
- list all artifact sources
- get artifacts source facts
- list ARM Template facts
- get ARM Template facts
- create instance of DevTest Lab Environment
- create instance of DevTest Lab Image
- delete instance of Lab

## Running the sample


## Variables

{% raw %}
```yaml
  vars:
    resource_group: "{{ resource_group_name }}"
    lab_name: myLab
    vn_name: myLabVirtualNetwork
    vm_name: myLabVm
    artifacts_name: myArtifacts
    github_token: "{{ lookup('env','GITHUB_ACCESS_TOKEN') }}"
    location: eastus
```
{% endraw %}

Following variables can be set in **vars** section of the playbook:

|Variable Name|Description|Notes|
|-------------|-----------|-----|
|resource_group|resource group where resources will be created|by default it's using **resource_group_name** parameter passed when running the playbook|
|location|location where resources should be created||
|lab_name|name of the lab instance||
|vn_name|lab virtual network name||
|vm_name|name of the vm instance||
|artifacts_name|lab artifacts name||
|github_token|GitHub token to access artifacts sources|by default playbook will attempt to get it from GITHUB_ACCESS_TOKEN environment variable|

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

## Creating DevTest Lab

This task creates an instance of DevTest Lab.

{% raw %}
```yaml
    - name: Create instance of Lab
      azure_rm_devtestlab:
        resource_group: "{{ resource_group }}"
        name: "{{ lab_name }}"
        location: "{{ location }}"
        storage_type: standard
        premium_data_disks: no
      register: output_lab
```
{% endraw %}

## DevTest Lab Policies

This task shows how to set up DevTest Lab policy settings.

Following values can be set:

|Value|Description|
|-----|-----------|
|user_owned_lab_vm_count|count of VMs that can be owned by an user|
|user_owned_lab_premium_vm_count|count of premium VMs that can be owned by an user|
|lab_vm_count|maximum lab VM count|
|lab_premium_vm_count|maximum lab premium VM count|
|lab_vm_size|allowed lab VMs size(s)|
|gallery_image|allowed gallery image(s)|
|user_owned_lab_vm_count_in_subnet|maximum number of user's VMs in a subnet|
|lab_target_cost|target cost of the lab|

{% raw %}
```yaml
    - name: Create instance of DevTest Lab Policy
      azure_rm_devtestlabpolicy:
        resource_group: "{{ resource_group }}"
        lab_name: "{{ lab_name }}"
        policy_set_name: myDtlPolicySet
        name: myDtlPolicy
        fact_name: user_owned_lab_vm_count
        threshold: 5
```
{% endraw %}

## DevTest Lab Schedules

Sample task setting DevTest Lab VM schedule.

Currently allowed are:
|Name|Description|
|----|-----------|
|lab_vms_startup|Lab VMs startup time|
|lab_vms_shutdown|Lab VMs shutdown time|

Time and time zone id must be specified.

{% raw %}
```yaml

    - name: Create instance of DevTest Lab Schedule
      azure_rm_devtestlabschedule:
        resource_group: "{{ resource_group }}"
        lab_name: "{{ lab_name }}"
        name: lab_vms_shutdown
        time: "1030"
        time_zone_id: "UTC+12"
      register: output
```
{% endraw %}

## DevTest Lab Virtual Network

This task creates default DevTest Lab virtual network.
{% raw %}
```yaml

    - name: Create instance of DevTest Labs virtual network
      azure_rm_devtestlabvirtualnetwork:
        resource_group: "{{ resource_group }}"
        lab_name: "{{ lab_name }}"
        name: "{{ vn_name }}"
        location: "{{ location }}"
        description: My DevTest Lab Virtual Network
      register: output
```
{% endraw %}

## DevTest Lab Artifact Source

This task shows how to create DevTest Lab artifacts source.
DevTest Lab artifacts source is properly structured GitHub repository that contains artifact definition and ARM templates.
Please note that every lab comes with predefined public artifacts source.

{% raw %}
```yaml

    - name: Create instance of DevTest Labs artifacts source
      azure_rm_devtestlabartifactsource:
        resource_group: "{{ resource_group }}"
        lab_name: "{{ lab_name }}"
        name: "{{ artifacts_name }}"
        uri: https://github.com/Azure/azure_preview_modules.git
        source_type: github
        folder_path: /tasks
        security_token: "{{ github_token }}"
```
{% endraw %}

## DevTest Lab Virtual Machine

This task shows how to create DevTest Labs virtual machine.

{% raw %}
```yaml
    - name: Create instance of DTL Virtual Machine
      azure_rm_devtestlabvirtualmachine:
        resource_group: "{{ resource_group }}"
        lab_name: "{{ lab_name }}"
        name: "{{ vm_name }}"
        notes: Virtual machine notes, just something....
        os_type: linux
        vm_size: Standard_A2_v2
        user_name: dtladmin
        password: ZSasfovobocu$$21!
        lab_subnet:
          virtual_network_name: "{{ vn_name }}"
          name: "{{ vn_name }}Subnet"
        disallow_public_ip_address: no
        image:
          offer: UbuntuServer
          publisher: Canonical
          sku: 16.04-LTS
          os_type: Linux
          version: latest
        artifacts:
          - source_name: "{{ artifacts_name }}"
            source_path: "/Artifacts/linux-install-mongodb"
        allow_claim: no
        expiration_date: "2029-02-22T01:49:12.117974Z"
```
{% endraw %}

## Listing All Artifact Sources and Artifacts

This task lists all artifacts sources in the lab.
Please use it to see default and custom artifacts sources.

{% raw %}
```yaml

    - name: List all artifact sources
      azure_rm_devtestlabartifactsource_facts:
        resource_group: "{{ resource_group }}"
        lab_name: "{{ lab_name }}"
      register: output
    - debug:
        var: output
```
{% endraw %}

Following task lists all the artifact in **public repo** which is predefined artifact source.

{% raw %}
```yaml

    - name: Get artifacts source facts
      azure_rm_devtestlabartifact_facts:
        resource_group: "{{ resource_group }}"
        lab_name: "{{ lab_name }}"
        artifact_source_name: public repo
      register: output
    - debug:
        var: output
```
{% endraw %}

## Getting Information on ARM Templates in Artifact Source

This task lists all the ARM templates in **public environment repo** that is predefined repo with templates.

{% raw %}
```yaml

    - name: List ARM Template facts
      azure_rm_devtestlabarmtemplate_facts:
        resource_group: "{{ resource_group }}"
        lab_name: "{{ lab_name }}"
        artifact_source_name: "public environment repo"
      register: output
    - debug:
        var: output
```
{% endraw %}

And following task retrieves details of a specific ARM template from the repository: 

{% raw %}
```yaml

    - name: Get ARM Template facts
      azure_rm_devtestlabarmtemplate_facts:
        resource_group: "{{ resource_group }}"
        lab_name: "{{ lab_name }}"
        artifact_source_name: "public environment repo"
        name: ServiceFabric-LabCluster
      register: output
    - debug:
        var: output
```
{% endraw %}

## Creating DevTest Lab Environment

Finally following task creates DevTest Lab environment. As you see it refers one of the templates from **public environment repo**.

{% raw %}
```yaml

    - name: Create instance of DevTest Lab Environment
      azure_rm_devtestlabenvironment:
        resource_group: "{{ resource_group }}"
        lab_name: "{{ lab_name }}"
        user_name: "@me"
        name: myEnvironment
        location: eastus
        deployment_template: "{{ output_lab.id }}/artifactSources/public environment repo/armTemplates/WebApp"
      register: output
```
{% endraw %}

## Creating DevTest Lab Image

This is also a very useful task. It creates DevTest Lab image from existing DevTest Lab Virtual Machine.
It can be later used to created new DevTest Lab virtual machines.

{% raw %}
```yaml

    - name: Create instance of DevTest Lab Image
      azure_rm_devtestlabcustomimage:
        resource_group: "{{ resource_group }}"
        lab_name: "{{ lab_name }}"
        name: myImage
        source_vm: "{{ output_vm.virtualmachines[0]['name'] }}"
        linux_os_state: non_deprovisioned
```
{% endraw %}

## Deleting the Lab

Final task deletes entire lab.

{% raw %}
```yaml
    - name: Delete instance of Lab
      azure_rm_devtestlab:
        resource_group: "{{ resource_group }}"
        name: "{{ lab_name }}"
        state: absent
      register: output
    - name: Assert the change was correctly reported
      assert:
        that:
          - output.changed
```
{% endraw %}
