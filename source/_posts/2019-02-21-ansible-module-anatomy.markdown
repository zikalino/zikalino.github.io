---
layout: post
title: "Ansible Module Anatomy"
date: 2019-02-21 12:27:21 +0800
comments: true
categories: 
---

This post describes what and where needs to be added in order to create a new module.

### File Locations

Let's say you want to create modules for a new Azure resource. You will need two modules - main module and facts module.
Following files should be created in Ansible directory:

```
~/lib/ansible/modules/cloud/azure/azure_rm_xxxxx.py
~/lib/ansible/modules/cloud/azure/azure_rm_xxxxx_facts.py
```

In addition to modules you need to create integration test.
In order to do that you will have to create following files:

```
~/test/integration/targets/azure_rm_xxxx/meta/main.yml
~/test/integration/targets/azure_rm_xxxx/tasks/main.yml
~/test/integration/targets/azure_rm_xxxx/aliases
```

Please note that integration tests can be shared (and in many cases should be shared) by multiple modules.

# Typical Module Dissected

## Header

Module header is always the same. Only parts that are changed are:
- copyright year
- original contributor name
- original contributor git handle

```python
#!/usr/bin/python
#
# Copyright (c) 2019 John Doe, (@johndoe)
#
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)

from __future__ import absolute_import, division, print_function
__metaclass__ = type


ANSIBLE_METADATA = {'metadata_version': '1.1',
                    'status': ['preview'],
                    'supported_by': 'community'}
```

## Documentation

The next section is the documentation

```python
DOCUMENTATION = '''
---
module: azure_rm_devtestlabvirtualmachine
version_added: "2.8"
short_description: Manage Azure DevTest Lab Virtual Machine instance.
description:
    - Create, update and delete instance of Azure DevTest Lab Virtual Machine.

options:
    resource_group:
        description:
            - The name of the resource group.
        required: True
    lab_name:
        description:
            - The name of the lab.
        required: True
    name:
        description:
            - The name of the virtual machine.
        required: True
    notes:
        description:
            - The notes of the virtual machine.
    state:
      description:
        - Assert the state of the Virtual Machine.
        - Use 'present' to create or update an Virtual Machine and 'absent' to delete it.
      default: present
      choices:
        - absent
        - present

extends_documentation_fragment:
    - azure
    - azure_tags

author:
    - "Zim Kalinowski (@zikalino)"
'''
```

## Examples

```python
EXAMPLES = '''
  - name: Create (or update) Virtual Machine
    azure_rm_devtestlabvirtualmachine:
      resource_group: myrg
      lab_name: mylab
      name: myvm
      notes: Virtual machine notes....
      os_type: linux
      vm_size: Standard_A2_v2
      user_name: vmadmin
      password: ZSuppas$$21!
      lab_subnet:
        name: myvnSubnet
        virtual_network_name: myvn
      disallow_public_ip_address: no
      image:
        offer: UbuntuServer
        publisher: Canonical
        sku: 16.04-LTS
        os_type: Linux
        version: latest
      artifacts:
        - source_name: myartifact
          source_path: "/Artifacts/linux-install-mongodb"
      allow_claim: no
      expiration_date: "2019-02-22T01:49:12.117974Z"
'''
```

## Return Value

Return value should contain detailed specification of return value in YAML format.

Please note that the main module should:
- contain resource id
- contain information that is essential for subsequent use of the resource (for instance FQDN, IP address, etc.)
- shouldn't contain information that is supplied via input parameters


```python
RETURN = '''
id:
    description:
        - The identifier of the DTL Virtual Machine resource.
    returned: always
    type: str
    sample: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myrg/providers/microsoft.devtestlab/labs/mylab/virtualmachines/myvm
compute_id:
    description:
        - The identifier of the underlying Compute Virtual Machine resource.
    returned: always
    type: str
    sample: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myrg/providers/microsoft.devtestlab/labs/mylab/virtualmachines/myvm
fqdn:
    description:
        - Fully qualified domain name or IP Address of the virtual machine.
    returned: always
    type: str
    sample: myvm.eastus.cloudapp.azure.com
'''
```

## Imports

Usually very similar, may contain additional imports related to particular Azure resource, etc.

```python
import time
from ansible.module_utils.azure_rm_common import AzureRMModuleBase
from ansible.module_utils.common.dict_transformations import _snake_to_camel

try:
    from msrestazure.azure_exceptions import CloudError
    from msrest.polling import LROPoller
    from msrestazure.azure_operation import AzureOperationPoller
    from azure.mgmt.devtestlabs import DevTestLabsClient
    from msrest.serialization import Model
except ImportError:
    # This is handled in azure_rm_common
    pass
```

## Class Definition and Init Function

Action types definition:

```python
class Actions:
    NoAction, Create, Update, Delete = range(4)
```

```python
class AzureRMVirtualMachine(AzureRMModuleBase):
    """Configuration class for an Azure RM Virtual Machine resource"""
    def __init__(self):
```

### Module Argument Specification

``` python
        self.module_arg_spec = dict(
            resource_group=dict(
                type='str',
                required=True
            ),
            lab_name=dict(
                type='str',
                required=True
            ),
            name=dict(
                type='str',
                required=True
            ),
            notes=dict(
                type='str'
            )
        )

```

### Conditional Required Options
``` python
        required_if = [
            ('state', 'present', [
             'image', 'lab_subnet', 'vm_size', 'os_type'])
        ]
```

### Argument Properties

``` python
        self.resource_group = None
        self.lab_name = None
        self.name = None
        self.lab_virtual_machine = dict()
```

### Module State

``` python
        self.results = dict(changed=False)
        self.mgmt_client = None
        self.state = None
        self.to_do = Actions.NoAction

```

### Base Class Initialization

``` python
        super(AzureRMVirtualMachine, self).__init__(derived_arg_spec=self.module_arg_spec,
                                                    supports_check_mode=True,
                                                    supports_tags=True,
                                                    required_if=required_if)
```

### Main Module Execution Method

```python
    def exec_module(self, **kwargs):
        """Main module execution method"""
```

### Mapping parameters to class properties:

```python
        for key in list(self.module_arg_spec.keys()) + ['tags']:
            if hasattr(self, key):
                setattr(self, key, kwargs[key])
            elif kwargs[key] is not None:
                self.lab_virtual_machine[key] = kwargs[key]
```

### Parameter transformations

Notes:
- we have several approaches right now
- in general here input parameters structure should be converted to match SDK call parameters

```python
        self.lab_virtual_machine['gallery_image_reference'] = self.lab_virtual_machine.pop('image', None)

        if self.lab_virtual_machine.get('artifacts') is not None:
            for artifact in self.lab_virtual_machine.get('artifacts'):
                source_name = artifact.pop('source_name')
                source_path = artifact.pop('source_path')
                template = "/subscriptions/{0}/resourceGroups/{1}/providers/Microsoft.DevTestLab/labs/{2}/artifactsources/{3}{4}"
                artifact['artifact_id'] = template.format(self.subscription_id, self.resource_group, self.lab_name, source_name, source_path)

        self.lab_virtual_machine['size'] = self.lab_virtual_machine.pop('vm_size')
        self.lab_virtual_machine['os_type'] = _snake_to_camel(self.lab_virtual_machine['os_type'], True)

        if self.lab_virtual_machine.get('storage_type'):
            self.lab_virtual_machine['storage_type'] = _snake_to_camel(self.lab_virtual_machine['storage_type'], True)

        lab_subnet = self.lab_virtual_machine.pop('lab_subnet')

        if isinstance(lab_subnet, str):
            vn_and_subnet = lab_subnet.split('/subnets/')
            if (len(vn_and_subnet) == 2):
                self.lab_virtual_machine['lab_virtual_network_id'] = vn_and_subnet[0]
                self.lab_virtual_machine['lab_subnet_name'] = vn_and_subnet[1]
            else:
                self.fail("Invalid 'lab_subnet' resource id format")
        else:
            template = "/subscriptions/{0}/resourceGroups/{1}/providers/Microsoft.DevTestLab/labs/{2}/virtualnetworks/{3}"
            self.lab_virtual_machine['lab_virtual_network_id'] = template.format(self.subscription_id,
                                                                                 self.resource_group,
                                                                                 self.lab_name,
                                                                                 lab_subnet.get('virtual_network_name'))
            self.lab_virtual_machine['lab_subnet_name'] = lab_subnet.get('name')
```

### Create Mgmt Client Instance

```python
        response = None

        self.mgmt_client = self.get_mgmt_svc_client(DevTestLabsClient,
                                                    base_url=self._cloud_environment.endpoints.resource_manager)

```

### Getting old response and idempotency check

```python
        old_response = self.get_virtualmachine()

        if not old_response:
            self.log("Virtual Machine instance doesn't exist")
            if self.state == 'absent':
                self.log("Old instance didn't exist")
            else:
                self.to_do = Actions.Create
            # get location from the lab as it has to be the same and has to be specified (why??)
            lab = self.get_devtestlab()
            self.lab_virtual_machine['location'] = lab['location']
        else:
            self.log("Virtual Machine instance already exists")
            if self.state == 'absent':
                self.to_do = Actions.Delete
            elif self.state == 'present':
                self.lab_virtual_machine['location'] = old_response['location']

                if old_response['size'].lower() != self.lab_virtual_machine.get('size').lower():
                    self.lab_virtual_machine['size'] = old_response['size']
                    self.module.warn("Property 'size' cannot be changed")

                if self.lab_virtual_machine.get('storage_type') is not None and \
                   old_response['storage_type'].lower() != self.lab_virtual_machine.get('storage_type').lower():
                    self.lab_virtual_machine['storage_type'] = old_response['storage_type']
                    self.module.warn("Property 'storage_type' cannot be changed")

                if old_response.get('gallery_image_reference', {}) != self.lab_virtual_machine.get('gallery_image_reference', {}):
                    self.lab_virtual_machine['gallery_image_reference'] = old_response['gallery_image_reference']
                    self.module.warn("Property 'image' cannot be changed")

                # currently artifacts can be only specified when vm is created
                # and in addition we don't have detailed information, just a number of "total artifacts"
                if len(self.lab_virtual_machine.get('artifacts', [])) != old_response['artifact_deployment_status']['total_artifacts']:
                    self.module.warn("Property 'artifacts' cannot be changed")

                if self.lab_virtual_machine.get('disallow_public_ip_address') is not None:
                    if old_response['disallow_public_ip_address'] != self.lab_virtual_machine.get('disallow_public_ip_address'):
                        self.module.warn("Property 'disallow_public_ip_address' cannot be changed")
                self.lab_virtual_machine['disallow_public_ip_address'] = old_response['disallow_public_ip_address']

                if self.lab_virtual_machine.get('allow_claim') is not None:
                    if old_response['allow_claim'] != self.lab_virtual_machine.get('allow_claim'):
                        self.module.warn("Property 'allow_claim' cannot be changed")
                self.lab_virtual_machine['allow_claim'] = old_response['allow_claim']

                if self.lab_virtual_machine.get('notes') is not None:
                    if old_response['notes'] != self.lab_virtual_machine.get('notes'):
                        self.to_do = Actions.Update
                else:
                    self.lab_virtual_machine['notes'] = old_response['notes']

```

### Performing Desired Action

```python
        if (self.to_do == Actions.Create) or (self.to_do == Actions.Update):
            self.log("Need to Create / Update the Virtual Machine instance")

            self.results['changed'] = True
            if self.check_mode:
                return self.results

            response = self.create_update_virtualmachine()

            self.log("Creation / Update done")
        elif self.to_do == Actions.Delete:
            self.log("Virtual Machine instance deleted")
            self.results['changed'] = True

            if self.check_mode:
                return self.results

            self.delete_virtualmachine()
        else:
            self.log("Virtual Machine instance unchanged")
            self.results['changed'] = False
            response = old_response
```

### Formatting Response

```python
        if self.state == 'present':
            self.results.update({
                'id': response.get('id', None),
                'compute_id': response.get('compute_id', None),
                'fqdn': response.get('fqdn', None)
            })
        return self.results
```

## Create / Update Function

```python
    def create_update_virtualmachine(self):
        '''
        Creates or updates Virtual Machine with the specified configuration.

        :return: deserialized Virtual Machine instance state dictionary
        '''
        self.log("Creating / Updating the Virtual Machine instance {0}".format(self.name))

        try:
            response = self.mgmt_client.virtual_machines.create_or_update(resource_group_name=self.resource_group,
                                                                          lab_name=self.lab_name,
                                                                          name=self.name,
                                                                          lab_virtual_machine=self.lab_virtual_machine)
            if isinstance(response, LROPoller) or isinstance(response, AzureOperationPoller):
                response = self.get_poller_result(response)

        except CloudError as exc:
            self.log('Error attempting to create the Virtual Machine instance.')
            self.fail("Error creating the Virtual Machine instance: {0}".format(str(exc)))
        return response.as_dict()
```

## Delete Function

```python
    def delete_virtualmachine(self):
        '''
        Deletes specified Virtual Machine instance in the specified subscription and resource group.

        :return: True
        '''
        self.log("Deleting the Virtual Machine instance {0}".format(self.name))
        try:
            response = self.mgmt_client.virtual_machines.delete(resource_group_name=self.resource_group,
                                                                lab_name=self.lab_name,
                                                                name=self.name)
        except CloudError as e:
            self.log('Error attempting to delete the Virtual Machine instance.')
            self.fail("Error deleting the Virtual Machine instance: {0}".format(str(e)))

        if isinstance(response, LROPoller) or isinstance(response, AzureOperationPoller):
            response = self.get_poller_result(response)

        return True
```

## Get Function

```python
    def get_virtualmachine(self):
        '''
        Gets the properties of the specified Virtual Machine.

        :return: deserialized Virtual Machine instance state dictionary
        '''
        self.log("Checking if the Virtual Machine instance {0} is present".format(self.name))
        found = False
        try:
            response = self.mgmt_client.virtual_machines.get(resource_group_name=self.resource_group,
                                                             lab_name=self.lab_name,
                                                             name=self.name)
            found = True
            self.log("Response : {0}".format(response))
            self.log("Virtual Machine instance : {0} found".format(response.name))
        except CloudError as e:
            self.log('Did not find the Virtual Machine instance.')
        if found is True:
            return response.as_dict()

        return False
```

## Module Entry Point

Always the same and looks as follows:

```python
def main():
    """Main execution"""
    AzureRMVirtualMachine()


if __name__ == '__main__':
    main()
```
