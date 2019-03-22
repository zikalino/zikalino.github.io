---
layout: post
title: "Ansible and Azure Function Apps"
date: 2019-03-07 22:07:46 +0800
comments: true
categories: 
---

This is something I was planning to investigate for some time. We have **azure_rm_functionapp** module, but it seems to be very limited. Actually I don't really know what it creates.

I decided to reverse engineer a very simple scenario - creating a container based function app through Azure Portal.

The best way to figure out what was created is to use following playbook to list all the resources in the resource group:

```yaml
    - name: List all the resources in the resource group
      azure_rm_resource_facts:
        api_version: '2018-05-01'
        resource_group: zimsfunctionapp
        resource_type: resources
      register: output
    - debug:
        var: output
```

Inside response I found following list of resources:

```json
[
    {
        "id": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimsfunctionapp/providers/Microsoft.Web/sites/zimsfunctionapp",
        "kind": "functionapp,linux,container",
        "location": "westus",
        "name": "zimsfunctionapp",
        "type": "Microsoft.Web/sites"
    }
]
```

Now I can do another query to get more details:

```yaml
    - name: Get WebApp information
      azure_rm_resource_facts:
        api_version: '2016-08-01'
        provider: web
        resource_group: zimsfunctionapp
        resource_type: sites
        resource_name: zimsfunctionapp
      register: output
    - debug:
        var: output
```

And that returs following large structure:

```json
[
    {
        "id": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/zimsfunctionapp/providers/Microsoft.Web/sites/zimsfunctionapp",
        "kind": "functionapp,linux,container",
        "location": "West US",
        "name": "zimsfunctionapp",
        "properties": {
            "adminEnabled": true,
            "availabilityState": "Normal",
            "cers": null,
            "clientAffinityEnabled": true,
            "clientCertEnabled": false,
            "clientCertExclusionPaths": null,
            "cloningInfo": null,
            "computeMode": null,
            "containerSize": 1536,
            "contentAvailabilityState": "Normal",
            "csrs": [],
            "dailyMemoryTimeQuota": 0,
            "defaultHostName": "zimsfunctionapp.azurewebsites.net",
            "deploymentId": "zimsfunctionapp",
            "domainVerificationIdentifiers": null,
            "enabled": true,
            "enabledHostNames": [
                "zimsfunctionapp.azurewebsites.net",
                "zimsfunctionapp.scm.azurewebsites.net"
            ],
            "functionExecutionUnitsCache": null,
            "geoDistributions": null,
            "homeStamp": "waws-prod-bay-081",
            "hostNameSslStates": [
                {
                    "hostType": "Standard",
                    "ipBasedSslResult": null,
                    "ipBasedSslState": "NotConfigured",
                    "name": "zimsfunctionapp.azurewebsites.net",
                    "sslState": "Disabled",
                    "thumbprint": null,
                    "toUpdate": null,
                    "toUpdateIpBasedSsl": null,
                    "virtualIP": null
                },
                {
                    "hostType": "Repository",
                    "ipBasedSslResult": null,
                    "ipBasedSslState": "NotConfigured",
                    "name": "zimsfunctionapp.scm.azurewebsites.net",
                    "sslState": "Disabled",
                    "thumbprint": null,
                    "toUpdate": null,
                    "toUpdateIpBasedSsl": null,
                    "virtualIP": null
                }
            ],
            "hostNames": [
                "zimsfunctionapp.azurewebsites.net"
            ],
            "hostNamesDisabled": false,
            "hostingEnvironment": null,
            "hostingEnvironmentId": null,
            "hostingEnvironmentProfile": null,
            "httpsOnly": false,
            "hyperV": false,
            "inProgressOperationId": null,
            "isXenon": false,
            "kind": "functionapp,linux,container",
            "lastModifiedTimeUtc": "2019-03-07T13:52:14.7033333",
            "maxNumberOfWorkers": null,
            "name": "zimsfunctionapp",
            "outboundIpAddresses": "13.64.73.110,40.118.133.8,40.118.169.141,40.118.253.162,13.64.147.140",
            "owner": null,
            "possibleOutboundIpAddresses": "13.64.73.110,40.118.133.8,40.118.169.141,40.118.253.162,13.64.147.140,52.160.85.217,13.93.238.69",
            "redundancyMode": "None",
            "repositorySiteName": "zimsfunctionapp",
            "reserved": true,
            "resourceGroup": "zimsfunctionapp",
            "runtimeAvailabilityState": "Normal",
            "scmSiteAlsoStopped": false,
            "selfLink": "https://waws-prod-bay-081.api.azurewebsites.windows.net:454/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/webspaces/jw-webapp-linux-nginx-06-WestUSwebspace/sites/zimsfunctionapp",
            "serverFarm": null,
            "serverFarmId": "/subscriptions/1c5b82ee-9294-4568-b0c0-b9c523bc0d86/resourceGroups/jw-webapp-linux-nginx-06/providers/Microsoft.Web/serverfarms/jw-webapp-linux-plan-01",
            "siteConfig": null,
            "siteDisabledReason": 0,
            "siteMode": null,
            "siteProperties": {
                "appSettings": null,
                "metadata": null,
                "properties": [
                    {
                        "name": "LinuxFxVersion",
                        "value": "DOCKER|httpd"
                    },
                    {
                        "name": "WindowsFxVersion",
                        "value": null
                    }
                ]
            },
            "sku": "Standard",
            "slotSwapStatus": null,
            "sslCertificates": null,
            "state": "Running",
            "storageRecoveryDefaultState": "Running",
            "suspendedTill": null,
            "tags": null,
            "targetSwapSlot": null,
            "trafficManagerHostNames": null,
            "usageState": "Normal",
            "webSpace": "jw-webapp-linux-nginx-06-WestUSwebspace"
        },
        "type": "Microsoft.Web/sites"
    }
]
```
