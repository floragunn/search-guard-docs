---
title: Activate and deactivate watch
html_title: Activating and deactivating a watch with the REST API
slug: elasticsearch-alerting-rest-api-watch-activate
category: signals-rest
order: 500
layout: docs
edition: preview
description: 
---

<!--- Copyright 2019 floragunn GmbH -->

# Activate/Deactivate Watch API
{: .no_toc}

{% include toc.md %}


## Endpoint

```
PUT /_signals/watch/{tenant_id}/{watch_id}/_activate
```

```
PUT /_signals/watch/{tenant_id}/{watch_id}/_deactivate
```

These endpoints can be used to activate and deactivate watches. Inactive watches are not automatically executed.

## Path Parameters

**{tenant_id}** The Signals tenant to be used. Specify `_main` to select the default tenant. If multi tenancy is disabled, `_main` is the only possible value.

**{watch_id}** The id of the watch to be activated or deactivated. Required.

## Request Body

No request body is required for this endpoint.

## Responses

### 200 OK

A watch identified by the given id exists and was successfully activated or deactivated.

### 403 Forbidden

The user does not have the permission to activate or deactivate watches for the currently selected tenant. 

### 404 Not found

A watch with the given id does not exist for the current tenant.

## Permissions

For being able to access the endpoint, the user needs to have the privilege `cluster:admin:searchguard:tenant:signals:watch/activate_deactivate` for the currently selected tenant.

This permission is distinct for the permission required to create or updated watches. Thus, a user may be allowed to activate or deactivate watches without being allowed to create or update watches.

This permission is included in the following [built-in action groups](security_permissions.md):

* SGS\_SIGNALS\_WATCH\_ACTIVATE

## Examples

### Basic 

```
PUT /_signals/watch/_main/bad_weather/_deactivate
```

**Response**

```
200 OK
``` 

