---
layout: post
title: "Waiting for Resources in Ansible"
date: 2019-01-25 07:08:54 +0800
comments: true
categories: 
---

Some resources may be not in a desired state after running a playbook. Especially when using Azure REST API, resource may be in **Creating** or **Provisioning** state. Sometimes it's necessary to wait unll provisioning state becomes **Succeeded**.

The best way to do this is to use **facts** module to periodically query the resource and exit the loop when desired state is achieved. Here's how to do it.

After the task that creates your virtual machine (or later), put following tasks. Actually only **Wait for Virtual Machine to be ready** is essential here. All the other tasks are just for demo purposes.

```yaml
    - name: Get VM facts
      azure_rm_resource_facts:
        api_version: '2018-10-01'
        resource_group: '{{ resource_group }}'
        provider: Compute
        resource_type: virtualMachines
        resource_name: "{{ virtual_machine_name }}"
      register: output

    - debug:
        var: output.response[0].properties.provisioningState

    - name: Wait for Virtual Machine to be ready
      azure_rm_resource_facts:
        api_version: '2018-10-01'
        resource_group: '{{ resource_group }}'
        provider: Compute
        resource_type: virtualMachines
        resource_name: "{{ virtual_machine_name }}"
      register: output
      until: output.response[0].properties.provisioningState == 'Succeeded'
      delay: 60
      retries: 5

    - debug:
        var: output.response[0].properties.provisioningState
```

You should see following output:

```
TASK [Create a vm with password authentication.] *******************************************
changed: [localhost]

TASK [Get VM facts] ************************************************************************
ok: [localhost]

TASK [debug] ******************************************************************************
ok: [localhost] => {
    "output.response[0].properties.provisioningState": "Creating"
}

TASK [Wait for Virtual Machine to be ready] *****************************************************
FAILED - RETRYING: Wait for Virtual Machine to be ready (5 retries left).
FAILED - RETRYING: Wait for Virtual Machine to be ready (4 retries left).
ok: [localhost]

TASK [debug] ******************************************************************************
ok: [localhost] => {
    "output.response[0].properties.provisioningState": "Succeeded"
}

PLAY RECAP ********************************************************************************
localhost                  : ok=18   changed=1    unreachable=0    failed=0    skipped=0
```

References:

Ansible documentation on do-until loops:
https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#do-until-loops
