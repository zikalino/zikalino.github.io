---
layout: post
title: "Ansible Modules - Proposed Improvements"
date: 2019-03-06 08:03:07 +0800
comments: true
categories: 
---

I have done a lot of experiments with Ansible module idempotence and here's a summary and a proposal of generic solution.

Let's start with problems we have:
- every module has their own idempotency checks so the behaviour is not consistent
- we agreed to display warning when change is detected but can't be applied. This is not implemented yet in majority in the modules.

## Manual Idempotence Check

- error prone
- time consuming
- no consistency between modules

## Generic Compare Function

- quite effective
- robust
- difficult to handle exceptions

### Comparison Types

|Type|Description|
|-|-|
|exact|This kind of comparison should work for most of the properties.|
|case insensitive|for instance resource id may need this type of comparison|
|location| location may be returned in different formats, for instance **east us** or **East US**|
|ignore|for properties that are not returned by GET, for instance passwords, secrets, or any other fields that we can't perform comparison|

### Exceptions

- sometimes (quite rarely) structure returned by GET is different from PUT
- some properties are not returned by GET, so can't compare
- sometimes comparison method needs to be adjusted to particular scenario

## Proposed Approach: Global Compare Function + Metadata in module_arg_spec

I have done appropriate tests and:
- it's possible to add custom metadata to **module_arg_spec** without disturbing Ansible sanity checks

Therefore we could have Azure specific metadata defined as follows, to provide auxilliary information for modules about:
- updatability of the field
- field relationship to structures obtained from **Get** and **CreateOrUpdate**
- idempotence comparison method

```python
self.module_arg_spec = dict(
    # ...
    option_a=dict(
        type='str',
        required=True,
        # Azure Module Specific Metadata
        updatable=False,
        disposition='parameters/option_a',
        comparison='insensitive'
    ),
)
```

|Name|Default|Description|
|-|-|-|
|updatable|**True**|Set to **False** if property cannot be updated. Warning will be automatically generated if change is detected.|
|disposition|-|Different name or path if property in different location after expanding options|
|comparison|**exact**|Comparison method, use one of: **exact**, **insensitive**, **location**, **ignore**|

Module base will provide function:

**map_and_compare(old_properties)**

**old_properties** is a dictionary returned by **Get** or empty dictionary **{}**.

Function will map new property values into existing dictionary or populate empty dictionary.

Function will return **True** if any change is detected.

Function will generate appropriate warnings if change of non-updatable fields is detected.
