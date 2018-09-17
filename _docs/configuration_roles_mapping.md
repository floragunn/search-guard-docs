---
title: Mapping Users to Roles
slug: mapping-users-roles
category: rolespermissions
order: 500
layout: docs
edition: community
description: How to map users and backend roles to Search Guard roles to implement flexible access control to an Elasticsearch cluster.
---
<!---
Copryight 2017 floragunn GmbH
-->
# Map users, backend roles and hosts to Search Guard roles

Hint: You can also use the [Kibana Confguration GUI](kibana_config_gui.md) for configuring the Roles Mapping.

Depending on your configuration, you can use the following data to assign a request to one or more Search Guard roles:

* username
  * the name of the authenticated user.
* backend roles
  * the roles fetched by the authorization backend(s), like LDAP, JWT or the internal user database.
* hostname / IP
  * the hostname or IP the request originated from.
* Common name
  * the DN of the client certificate sent with the request.

## Backend roles and Search Guard roles

Backend roles are roles that Search Guard retrieves during the authentication and authorization process. These roles are then mapped to the roles Search Guard uses to define which permissions a given user or host possesses. The permissions themselves can be defined in `sg_action_groups`, and the Search Guard (not backend) roles are defined in `sg_roles`, while `sg_roles_mapping` defined the connection between particular users and specific roles. 

Backend roles can come from:

* An LDAP server configured in the `authz` section of `sg_config.yml`
* Roles defined in `sg_internal_users.yml` for particular users
* A JSON web token, if you're using JWT authentication
* HTTP headers, if you're using Proxy authentication

## Mapping

Backend users, roles and hosts are mapped to Search Guard roles in the file `sg_roles_mapping.yml`.

Syntax:

```yaml
<Search Guard role name>:
  users:
    - <username>
    - ...
  backendroles:
    - <rolename>
    - ...
  hosts:
    - <hostname>
    - ...
```

**The Search Guard role name must not contain dots.**

Example:

```yaml
sg_read_write:
  users:
    - janedoe
    - johndoe
  backendroles:
    - management
    - operations
    - 'cn=ldaprole,ou=groups,dc=example,dc=com'
  hosts:
    - "*.devops.company.com"
```

A request can be assigned to one or more Search Guard roles. If a request is mapped to more than one role, the permissions of these roles are combined.

## Permission handling when assigning multiple roles 

A user can have as many roles as necessary, and all permissions for all roles are assigned to that user. However, if a user has multiple roles that define *different permissions for the same index*, then Search Guard will only use the permissions found in the first role.

If you would like to combine all permissions for that index, enable this feature in `sg_config.yml` like:

```
searchguard.dynamic.multi_rolespan_enabled: true
```

This will become the default behavior for Search Guard 7. At the moment the default for this switch is `false`for backwards compatibility.

## Advanced: Hostname lookup

Search Guard provides three different approaches to resolve the actual hostname against the configured hosts mapping in `sg_roles_mapping`. This can be configured in `sg_config.yml`:

```yaml
searchguard
  dynamic
    hosts_resolver_mode: <mode>
```

Where mode is one of:

| Name | Description |
|---|---|
| ip-only | Match IP addresses only. Default. |
| ip-hostname | Match IP addresses and hostnames |
| ip-hostname-lookup | Match IP addresses and hostnames, and perform a reverse hostname lookup |
