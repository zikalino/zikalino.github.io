---
layout: post
title: "Module azure_rm_virtualmachine - delta"
date: 2019-01-20 10:26:39 +0800
comments: true
categories: 
---

A

Below is a quick compilation and comparison:

|Feature|Idempotence|Support In Ansible|Idempotence in Ansible|
|-------|-----------|------------------|----------------------|
|identity||NO||
|location||YES||
|plan||YES||
|properties.additionalCapabilities.ultraSSDEnabled|deallocated only|NO||
|properties.availabilitySet.id|changes not allowed|||
|properties.diagnosticsProfile.bootDiagnostics.enabled||NO||
|properties.diagnosticsProfile.bootDiagnostics.storageUri||NO||
|properties.hardwareProfile.vmSize||YES|YES (check)|
|properties.licenseType||||
|properties.networkProfile.networkInterfaces[].id||YES|YES (check)|
|properties.osProfile.adminPassword|OK|||
|properties.osProfile.adminUsername|not allowed|||
|properties.osProfile.allowExtensionOperations|OK|||
|properties.osProfile.computerName|not allowed|YES|YES (check)|
|properties.osProfile.customData|not allowed|||
|properties.osProfile.linuxConfiguration.......||||
|properties.osProfile.secrets||||
|properties.osProfile.windowsConfiguration.......||||
|properties.osProfile.||||
|properties.storageProfile.dataDisks[].........||YES - check details||
|properties.storageProfile.imageReference|changes not allowed|||
|properties.storageProfile.osDisk...........||YES - check details||
|properties.storageProfile.osDisk.caching|OK|YES|YES (check)|
|properties.storageProfile.osDisk.name|not allowed|YES|YES (check)|
|properties.storageProfile.osDisk.diskSizeGB|deallocated only|YES|YES (check)|
|properties.storageProfile.osDisk.writeAcceleratorEnabled|managed only|||
|properties.storageProfile.osDisk.osType|changes not allowed|||
|properties.storageProfile.osDisk.createOption|changes not allowed|||
|tags||YES||
|zones||NO||

In addition REST API provides several APIs, some of them are supported, but some are not:

|Feature|Support In Ansible|
|-------|------------------|
|Capture|NO|
|Convert to Managed Disks||
|Deallocate|YES|
|Generalize|YES (2.8)|
|Perform Maintenance|NO|
|Power Off|YES|
|Redeploy|NO|
|Reimage|NO|
|Restart|YES|
|Start|YES|
