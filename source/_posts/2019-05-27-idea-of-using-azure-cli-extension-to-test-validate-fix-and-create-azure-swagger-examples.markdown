---
layout: post
title: "Idea of Using Azure CLI Extension to Test, Validate, Fix and Create Azure Swagger Examples"
date: 2019-05-27 11:01:54 +0800
comments: true
categories: 
---

## Overview

This idea occured to me for several reasons:

- there's no single tool used to create, test and validate Azure Swagger examples
- process of validating and updating Azure REST API examples is quite awkward and time consuming. Updating even single input value reauires a lot of steps, for instance responses should be updated as well
- there are no single naming conventions
- Azure CLI is the most commonly used tool to manage Azure resources
- it would be very easy to create an extension based on **az resource** which already allows raw REST API calls meaning it provides entire framework

## Advantages

- easy, fast and consistent process of creating new and updating existing examples
- validation and dependency checking
- enforcing consistent naming conventions, that would lead to more consistent documentation and compatibility between examples (for instance all the examples would use single resource group **myResourceGroup**)
- simple way to check examples without using any additional tool 

## Running / Validating Existing Examples

### Overview

The tool should allow to:

- run existing examples directly from Azure REST API specs (local or GitHub repo)
- run examples in their raw form
- automatically fix problems / adjust naming conventions to match common naming convention
- allow applying custom parameter fixes 
- generate final adjusted / updated example source code
- optionally copy example to original source tree or create a pull request in original GitHub repo (fork of Azure REST API specification)

### Command

To run existing example:

**az swagger run --example {location} [--raw] [--override {field}={value}] [--dependencies] [--update]**

where:

**--example**

{location} - either URL of example in GitHub or local path to example in cloned GitHub repo

**--raw**

If specified all field values from the example files will be used as is.

By default extension would override existing values using default resource naming conventions.

**--override**

{field} - field name or path to a field in 

**--dependencies**

The tool will attempt to create all the dependencies if they don't exist, e.g. resource group or parent resource (using appropriate examples from swagger).

If this option is note specified, the tool will display appropriate warnings. 

**--update**

If this option is specified extension will attempt to update existing sample in the local filesystem.
In the future it could create a branch and pull requests directly to the GitHub Repo.

The attempt will be made if:

- example was run successfully
- there are differences between existing example input parameters and overrided input parameters
- there is difference in return value specified and value actually returned

Sample input could look like that:

```
az swagger run --example https://github.com/Azure/azure-rest-api-specs/blob/master/specification/botservice/resource-manager/Microsoft.BotService/preview/2018-07-12/examples/CreateBot.json
```


Sample output could look like this:

```
-------- WARNINGS
- resourceGroupName "OneResourceGroup" was replaced by "myResourceGroup"
- resourceName "sampleBot" was replaced by "myBot"
-------- ORIGINAL EXAMPLE
{
  "parameters": {
    "subscriptionId": "subscription-id",
    "resourceGroupName": "OneResourceGroupName",
    "api-version": "2017-01-01",
    "resourceName": "samplebotname",
    "parameters": {
      "location": "West US",
      "sku": {
        "name": "S1"
      },
      "etag": "etag1",
      "tags": {
        "tag1": "value1",
        "tag2": "value2"
      },
      "name": "samplename",
      "type": "sampletype",
      "id": "someid",
      "kind": "sdk",
      "properties": {
        "description": "The description of the bot",
        "developerAppInsightKey": "appinsightskey",
        "developerAppInsightsApiKey": "appinsightsapikey",
        "developerAppInsightsApplicationId": "appinsightsappid",
        "displayName": "The Name of the bot",
        "endpoint": "http://mybot.coffee",
        "iconUrl": "http://myicon",
        "luisAppIds": [
          "luisappid1",
          "luisappid2"
        ],
        "luisKey": "luiskey",
        "msaAppId": "exampleappid"
      }
    }
  },
  "responses": {
    "200": {
      "body": {
        "location": "West US",
        "tags": {
          "tag1": "value1",
          "tag2": "value2"
        },
        "name": "samplename",
        "type": "sampletype",
        "id": "someid",
        "kind": "sdk",
        "etag": "etag1",
        "properties": {
          "description": "The description of the bot",
          "developerAppInsightKey": "appinsightskey",
          "developerAppInsightsApplicationId": "appinsightsappid",
          "displayName": "The Name of the bot",
          "endpoint": "http://mybot.coffee",
          "endpointVersion": "version",
          "iconUrl": "http://myicon",
          "luisAppIds": [
            "luisappid1",
            "luisappid2"
          ],
          "msaAppId": "msaappid",
          "configuredChannels": [
            "facebook",
            "groupme"
          ],
          "enabledChannels": [
            "facebook"
          ]
        }
      }
    },
    "201": {
      "body": {
        "location": "West US",
        "tags": {
          "tag1": "value1",
          "tag2": "value2"
        },
        "name": "samplename",
        "type": "sampletype",
        "id": "someid",
        "kind": "sdk",
        "properties": {
          "description": "The description of the bot",
          "developerAppInsightsApplicationId": "appinsightsappid",
          "displayName": "The Name of the bot",
          "endpoint": "http://mybot.coffee",
          "endpointVersion": "version",
          "iconUrl": "http://myicon",
          "luisAppIds": [
            "luisappid1",
            "luisappid2"
          ],
          "msaAppId": "msaappid",
          "configuredChannels": [
            "facebook",
            "groupme"
          ],
          "enabledChannels": [
            "facebook"
          ]
        }
      }
    }
  }
}
-------- UPDATED EXAMPLE (createbot.json)
{
  "parameters": {
    "subscriptionId": "subscription-id",
    "resourceGroupName": "myResourceGroup",
    "api-version": "2017-01-01",
    "resourceName": "myBot",
    "parameters": {
      "location": "West US",
      "sku": {
        "name": "S1"
      },
      "etag": "etag1",
      "tags": {
        "tag1": "value1",
        "tag2": "value2"
      },
      "name": "samplename",
      "type": "sampletype",
      "id": "someid",
      "kind": "sdk",
      "properties": {
        "description": "The description of the bot",
        "developerAppInsightKey": "appinsightskey",
        "developerAppInsightsApiKey": "appinsightsapikey",
        "developerAppInsightsApplicationId": "appinsightsappid",
        "displayName": "The Name of the bot",
        "endpoint": "http://mybot.coffee",
        "iconUrl": "http://myicon",
        "luisAppIds": [
          "luisappid1",
          "luisappid2"
        ],
        "luisKey": "luiskey",
        "msaAppId": "exampleappid"
      }
    }
  },
  "responses": {
    "200": {
      "body": {
        "location": "West US",
        "tags": {
          "tag1": "value1",
          "tag2": "value2"
        },
        "name": "samplename",
        "type": "sampletype",
        "id": "someid",
        "kind": "sdk",
        "etag": "etag1",
        "properties": {
          "description": "The description of the bot",
          "developerAppInsightKey": "appinsightskey",
          "developerAppInsightsApplicationId": "appinsightsappid",
          "displayName": "The Name of the bot",
          "endpoint": "http://mybot.coffee",
          "endpointVersion": "version",
          "iconUrl": "http://myicon",
          "luisAppIds": [
            "luisappid1",
            "luisappid2"
          ],
          "msaAppId": "msaappid",
          "configuredChannels": [
            "facebook",
            "groupme"
          ],
          "enabledChannels": [
            "facebook"
          ]
        }
      }
    },
    "201": {
      "body": {
        "location": "West US",
        "tags": {
          "tag1": "value1",
          "tag2": "value2"
        },
        "name": "samplename",
        "type": "sampletype",
        "id": "someid",
        "kind": "sdk",
        "properties": {
          "description": "The description of the bot",
          "developerAppInsightsApplicationId": "appinsightsappid",
          "displayName": "The Name of the bot",
          "endpoint": "http://mybot.coffee",
          "endpointVersion": "version",
          "iconUrl": "http://myicon",
          "luisAppIds": [
            "luisappid1",
            "luisappid2"
          ],
          "msaAppId": "msaappid",
          "configuredChannels": [
            "facebook",
            "groupme"
          ],
          "enabledChannels": [
            "facebook"
          ]
        }
      }
    }
  }
}
-------- RESULT
UPDATED: YES
RUN: SUCCESS
```

## Scaffolding Examples

### Overview

If example doesn't exist, the tool will allow creating a template based on REST API specs.
In the next step user should update the template and rerun the tool to validate example until it runs correctly.

### Command

**az swagger scaffold --swagger {location} --uri {resource-uri} --method {method} [--update]**

**--example**

{location} - either URL of specific swagger json file or path in local 

**--uri**

Resource URI as appears in swagger specification / documentation.

**--method**

Method for which scaffolding should be done, e.g. **put**, **get**, etc...

**--update**

If this option is specified extension will attempt to update existing sample in the local filesystem.
In the future it could create a branch and pull requests directly to the GitHub Repo.
