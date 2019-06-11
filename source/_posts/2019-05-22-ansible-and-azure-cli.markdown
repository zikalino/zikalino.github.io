---
layout: post
title: "Ansible and Azure CLI - Closer than I Thought"
date: 2019-05-22 11:25:08 +0800
comments: true
categories: 
---

### What if...

... Ansible support for Azure was based on **Azure CLI**?

## Similarities and Differences


|Item|Azure CLI|Ansible|
|----|---------|-------|
|Implementation|Python/Python SDK|Python/Python SDK|
|UX|Azure CLI Options are flattened, simplified and well designed|Ansible options are flattened and we strive to simplify the structure. The level of simplification of Azure CLI is our ultimate goal.|
|Error Handling|Excellent. Azure CLI has a lot of checks and gracefully handles critical situations. User can easily create any new resource by reading documentation and reading error responses when something goes wrong.|Poor, if user specifies wrong parameters in most cases module propagates error received from Azure, which in most cases is meaningless|
|Idempotence|Not a major goal of Azure CLI, however the idea is that Azure CLI commands should be idempotent|Idempotence is a major feature of Ansible, however we have several implementation problems, for instance not handling non-updatable fiels correctly, not detecting changes in parameters corectly. A lot of inconsistencies in the implementation|
|Documentation|Excellent|Good, but sometimes not complete or directly copied from REST API Specification|
|Check Mode|NO|YES (see notes on idempotence)|
|Coverage|Around 60% of Azure API|Around 12% of Azure API|

## How it Could Be Done?

Implementation could be extremely simple

We could have entire implementation in a single base **az.py** Ansible module to encapsulate **Azure CLI**.

Instead of implementing hundreds of modules for every single resource, we could just have symbolic links to the base module:

- **az_vm.py**
- **az_acr.py**
- **az_vmss.py**

Base module implementation knows via which link it was loaded, so based on that it can dynamically create:

- appropriate documentation
- appropriate argument spec
- handle arguments correctly and call correct commands

On the top of that module can:
- recognise which options are updatable by comparing **create** and **update** parameters (using distilled knowledge from Azure CLI)

### How module would work?

(1) call **az xxx get** to check whether resource exists, get current state
(2) call **az xxx create** or **az xxx update** if resource exists
(3) compare output of initial **get** with output of **update**, to check whether anything has changed

## What's missing?

### Check Mode

Check mode is probably the only missing feature that Ansible has (but we are struggling to implement it correctly) and Azure CLI doesn't.

As far as I know Azure CLI is striving to implement idempotency. I need to collect more details on that.

Anyway, a new option called **--check-mode** would be nice to have.
It could work as follows:

User calls:

**az xxxx create ..... params .....**

Azure CLI calls **get** method on the resource to be created.
Azure CLI prepares the structure to call Python SDK API.
Acure CLI compares structure returned by **get** and structure prepared for **create** call.
If the structures match, return **no change required**, if structures don't match return list of changes detected.

This method would probably work for around 90% of Azure resources, it would fail with 





This module would just handle all the interaction with unerlying **Azure CLI**, including:
- handling currently selected subscription if necessary
- handling authentication

On the top of that we could just create symbolic links to that module for every Azure resource, for instance:


**az** module knows through which link it was called, so it could generate documentation and argument specification dynamically.

### Benefits

- Azure CLI has extremely well maintained UX, which just appears ideal for tools like Ansible
- Examples are curated for every resource and well tested
- Documentation could be just reused without any changes

### What's missing?

- idempotence
- idempotence
- idempotence

