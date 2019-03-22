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



## Creating DevTest Lab

```yaml
    - name: Create instance of Lab
      azure_rm_devtestlab:
        resource_group: "{{ resource_group }}"
        name: "{{ lab_name }}"
        location: eastus
        storage_type: standard
        premium_data_disks: no
      register: output_lab
```

## DevTest Lab Policies

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

## DevTest Lab Schedules

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

## DevTest Lab Virtual Network

```yaml
    - name: Create instance of DevTest Labs virtual network
      azure_rm_devtestlabvirtualnetwork:
        resource_group: "{{ resource_group }}"
        lab_name: "{{ lab_name }}"
        name: "{{ vn_name }}"
        location: eastus
        description: My DevTest Lab
      register: output
```

## DevTest Lab Artifact Source

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

## DevTest Lab Virtual Machine

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

## Listing All Artifact Sources and Artifacts

```yaml
    - name: List all artifact sources
      azure_rm_devtestlabartifactsource_facts:
        resource_group: "{{ resource_group }}"
        lab_name: "{{ lab_name }}"
      register: output
    - debug:
        var: output
```

```yaml
    - name: Get artifacts source facts
      azure_rm_devtestlabartifactsource_facts:
        resource_group: "{{ resource_group }}"
        lab_name: "{{ lab_name }}"
        name: public repo
      register: output
    - debug:
        var: output
```

## Getting Information on ARM Templates in Artifact Source

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

## Creating DevTest Lab Environment

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

## Creating DevTest Lab Image

```yaml
    - name: Create instance of DevTest Lab Image
      azure_rm_devtestlabcustomimage:
        resource_group: "{{ resource_group }}"
        lab_name: "{{ lab_name }}"
        name: myImage
        source_vm: "{{ output_vm.virtualmachines[0]['name'] }}"
        linux_os_state: non_deprovisioned
```

## Deleting the Lab

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
