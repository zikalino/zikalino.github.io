---
layout: post
title: "Creating Container Based Function App Using Azure CLI"
date: 2019-03-27 12:53:55 +0800
comments: true
categories: 
---

It's just a few lines of code, but took me some time to do it by trial and error.

Just a few steps:

(1) create resource group

(2) create storage account

(3) create appservice plan with **is_linux** option

(4) create function app and specify container image **httpd** from DockerHub

```bash
az group create --name zimsfromclixx --location westeurope

az storage account create --name zimsstoragefromclixx --location westeurope --resource-group zimsfromclixx --sku Standard_LRS

az appservice plan create --name myplan -g zimsfromclixx --is-linux --location westeurope --sku S1 --number-of-workers 1

az functionapp create --resource-group zimsfromclixx --os-type Linux --plan myplan --name zimsfromclixx --deployment-container-image-name httpd --storage-account zimsstoragefromclixx
```

Just run the commands using Azure CLI, go to the browser and enter https://zimsfromclixx.azurewebsites.net/ (you may use your own functionapp name here), and you should see standard Apache's **It Works!!" welcome page.

Sounds easy, however when creating functionapp, **--plan** argument is not obligatory, and if you omit step (3) and **--plan** argument, default plan will be created, and with default plan you can't use containers. It took me some time to figure out....

Also some people run into similar problem, for instance here:

https://github.com/Azure/azure-cli/issues/7426

In fact only this PR helped me to understand how Azure CLI works:

https://github.com/Azure/azure-cli/pull/7435/files


When you try to specify **--deployment-container-image-name httpd** without passing **--plan** that is **is_linux**, Azure CLI resoponds with following message, which is quite misleading:

**usage error: --runtime RUNTIME required for linux functions apps without custom image.**

I submitted issue to Azure CLI regarding this: https://github.com/Azure/azure-cli/issues/8889

Finally, here's a link to some documentation I found later:

https://docs.microsoft.com/en-us/azure/app-service/containers/app-service-linux-cli
