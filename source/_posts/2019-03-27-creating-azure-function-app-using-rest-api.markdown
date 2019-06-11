---
layout: post
title: "Creating Azure Function App using REST API"
date: 2019-03-27 13:47:38 +0800
comments: true
categories: 
---

Finally I got it working...

I used appservice plan as created in my previous post:

```bash
az appservice plan create --name myplan -g zimsfromclixx --is-linux --location westeurope --sku S1 --number-of-workers 1
```

and updated my task to refer that Linux-enabled plan (**serverFarmId**):

```bash
    - name: Create Function App using REST API
      azure_rm_resource:
        api_version: '2018-02-01'
        resource_group: "{{ resource_group }}"
        provider: web
        resource_type: sites
        resource_name: "{{ fa_name }}"
        body:
          kind: functionapp,linux,container
          properties:
            siteConfig:
              appSettings:
                - name: FUNCTIONS_EXTENSION_VERSION
                  value': '~2'
                - name: DOCKER_REGISTRY_SERVER_URL
                  value: 'https://index.docker.io'
              linuxFxVersion: 'DOCKER|httpd'
            serverFarmId: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/zimsfromclixx/providers/Microsoft.Web/serverFarms/myplan
          location: eastus
```

Task is pretty simple, and I guess it may be simplified even further. I plan to create full sample, and add apropriate updates to **azure_rm_functionapp** module.
Plus I will create a new sample for Azure REST API specification. That was at least two days of work to get here....