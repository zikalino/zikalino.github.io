---
layout: post
title: "Coming Rename of _facts modules to _info modules"
date: 2019-03-25 13:24:19 +0800
comments: true
categories: 
---

It requires some statistics on what needs to be done:

|Module Name|Status|Action|
|-----------|------|------|
|azure_rm_aks_facts|2.6|** needs upgrade|
|azure_rm_applicationsecuritygroup_facts|2.8|rename|
|azure_rm_appserviceplan_facts|2.7|rename/obsolete|
|azure_rm_autoscale_facts|2.7|rename/obsolete|
|azure_rm_availabilityset_facts|OLD|** needs upgrade|
|azure_rm_cdnendpoint_facts|2.8|rename|
|azure_rm_cdnprofile_facts|2.8|rename|
|azure_rm_containerinstance_facts|2.8|rename|
|azure_rm_containerregistry_facts|2.7|rename/obsolete|
|azure_rm_cosmosdbaccount_facts|2.8|rename|
|azure_rm_deployment_facts|2.8|rename|
|azure_rm_devtestlabarmtemplate_facts|2.8|rename|
|azure_rm_devtestlabartifactsource_facts|2.8|rename|
|azure_rm_devtestlabartifact_facts|2.8|rename|
|azure_rm_devtestlabcustomimage_facts|2.8|rename|
|azure_rm_devtestlabenvironment_facts|2.8|rename|
|azure_rm_devtestlabartifact_facts|2.8|rename|
|azure_rm_devtestlabpolicy_facts|2.8|rename|
|azure_rm_devtestlabvirtualmachine_facts|2.8|rename|
|azure_rm_devtestlabvirtualnetwork_facts|2.8|rename|
|azure_rm_devtestlab_facts|2.8|rename|
|azure_rm_dnsrecordset_facts|OLD/SPLIT|split/obsolete|
|azure_rm_dnszone_facts|OLD/SPLIT|split/obsolete|
|azure_rm_functionapp_facts|OLD|** needs upgrade|
|azure_rm_hdinsight_facts|2.8|rename|
|azure_rm_image_facts|2.8|rename|
|azure_rm_loadbalancer_facts|OLD|** needs upgrade|
|azure_rm_managed_disk_facts|OLD|** needs upgrade|
|azure_rm_mariadbconfiguration_facts|2.8|rename|
|azure_rm_mariadbdatabase_facts|2.8|rename|
|azure_rm_mariadbfirewallrule_facts|2.8|rename|
|azure_rm_mariadbserver_facts|2.8|rename|
|azure_rm_mysqlconfiguration_facts|2.8|rename|
|azure_rm_mysqldatabase_facts||rename/obsolete|
|azure_rm_mysqlfirewallrule_facts|2.8|rename|
|azure_rm_mysqlserver_facts||rename/obsolete|
|azure_rm_networkinterface_facts|OLD/SPLIT|split/obsolete|
|azure_rm_postgresqlconfiguration_facts|2.8|rename|
|azure_rm_postgresqldatabase_facts||rename/obsolete|
|azure_rm_postgresqlfirewallrule_facts|2.8|rename|
|azure_rm_postgresqlserver_facts||rename/obsolete|
|azure_rm_publicipaddress_facts|OLD/SPLIT|split/obsolete|
|azure_rm_rediscache_facts|2.8|rename|
|azure_rm_resourcegroup_facts|OLD|** needs upgrade|
|azure_rm_resource_facts|2.6|rename/obsolete|
|azure_rm_roleassignment_facts|2.8|rename|
|azure_rm_roledefinition_facts|2.8|rename|
|azure_rm_routetable_facts|2.7|rename/obsolete|
|azure_rm_securitygroup_facts|OLD|** needs upgrade|
|azure_rm_sqldatabase_facts|2.8|rename|
|azure_rm_sqlfirewallrule_facts|2.8|rename|
|azure_rm_sqlserver_facts||** needs upgrade|
|azure_rm_subnet_facts|2.8|rename|
|azure_rm_storageaccount_facts|OLD/SPLIT?|split/obsolete|
|azure_rm_trafficmanagerendpoint_facts||rename/obsolete|
|azure_rm_trafficmanagerprofile_facts||rename/obsolete|
|azure_rm_virtualmachineextension_facts|2.8|rename|
|azure_rm_virtualmachineimage_facts|OLD|** needs upgrade|
|azure_rm_virtualmachinescalesetextension_facts|2.8|rename|
|azure_rm_virtualmachinescalesetinstance_facts|2.8|rename|
|azure_rm_virtualmachine_facts|2.7|rename/obsolete|
|azure_rm_virtualmachine_scaleset_facts|OLD/SPLIT|split/obsolete|
|azure_rm_virtualnetwork_facts|OLD/SPLIT|split/obsolete|
|azure_rm_webapp_facts|2.7|rename/obsolete|

From above:

**needs upgrade** - these modules need new *curated* version (9 modules)

**split/obsolete** - these modules alerady have *curated* format, need to be split (7 modules)

**rename/obsolete** - these modules were introduced before 2.8 and we need to rename them and made old version available via symbolic link and obsolete (14 modules)

**rename** - these modules we introduced in Ansible 2.8, meaning they are not release them, meaning we can release them before freeze (around 35 modules)

## Impact on Sample Playbooks

- we need to rename **facts** to **info** in all playbooks referring to new 2.8 facts modules
- we have more then 1 year timeframe for older **facts** modules as they will be marked as obsolete

## Impact on users

- for all the new 2.8 modules they will see only new **info** modules, so there's no impact
- for older **facts** modules they can use either **facts** or **info**, if they use **facts** they will be encouraged to switch to new **info** modules, also impact is minimal
- they will have a year to change

## Impact on test
- same tests executed on new **info** modules as old **facts* modules
- in most cases **facts** modules will be symbolic links to **info** modules, so content will be the same
- for oldest modules, we can disable tests for old **fact** or we can test both **facts** and **info**

## Impact on azure_preview_modules
- no impact, we will just copy all the updated from devel, as usual