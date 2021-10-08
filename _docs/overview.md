---
title: Main concepts
slug: main-concepts
category: quickstart
order: 100
layout: docs
description: How Search Guard extracts credentials from a request and how they are mapped to users, roles and permissions.
resources:
  - "search-guard-presentations#quickstart|Search Guard Quickstart and First Steps (presentation)"
  - "search-guard-presentations#architecture-request-flow|Architecture and Request Flow (presentation)"
  - "search-guard-presentations#configuration-basics|Configuration Basics (presentation)"

---

<!---
Copyright 2016 floragunn GmbH
-->

# Search Guard main concepts
{: .no_toc}

{% include_relative _includes/toc.md %}

Search Guard can be used to secure your Elasticsearch cluster by working with different industry standard authentication techniques, like Kerberos, LDAP / Active Directory, JSON web tokens, TLS certificates and Proxy authentication / SSO.

Regardless of what authentication method you use, the basic flow is as follows:

* A user wants to **access** an Elasticsearch cluster, for example by issuing  a simple query.
* Search Guard retrieves the user's **credentials** from the request
  * How the credentials are retrieved depends on the authentication method. For example, they can be extracted from HTTP Basic Authentication headers, from a JSON web token or from a Kerberos ticket.
* Search Guard **authenticates** the credentials against the configured authentication backend(s).  
* Search Guard **authorizes** the user by retrieving a list of the user's roles from the configured authorization backend
  * Roles retrieved from authorization backends are called **backend roles**. 
  * For example, roles can be fetched from LDAP/AD, from a JSON web token or from the Search Guard internal user database.
* Search Guard **maps** the user and backend roles to **Search Guard roles**.
* Search Guard determines the **permissions** associated with the Search Guard role and decides whether the action the user wants to perform is allowed or not.
* If your are using [Document- and Field-Level-Security](dlsfls_dls.md), you can also apply more fine grained permissions based on documents and individual fields.  

## Authentication flow

<p align="center">
<img src="authentication_flow.png" style="width: 40%" class="md_image"/>
</p>

## Credentials

In order to **identify** the user who wants to interact with the cluster, Search Guard first needs the user's **credentials**. 

What the term credential means depends on the technology you use for user identification. For example, if you use HTTP basic auth, then the credentials are username and password.

If you use a JSON web token, the credentials are contained in the token itself.

If you use TLS certificates for identifying clients, the credentials are the DN of the certificate.

A credential provider can either be **challenging** or **non-challenging**. A challenging provider actively asks the user for his or her credentials if they are not already present in the request. A common way is to display an HTTP basic auth dialogue. 

A non-challenging authenticator always assumes that the credentials are present in the request, and will not ask the user for it in case they are missing.

## Authentication (authc)

Search Guard then **authenticates** the credentials **against a backend authentication module**, for example LDAP, Active Directory, Kerberos or JWT.

Search Guard supports chaining multiple authenticators. If more than one authenticator is configured, Search Guard will try to authenticate the user against all authenticators in the chain, until one authenticator succeeds. 

A common use case is to combine the Search Guard internal user database with LDAP / Active Directory.

### External authentication / SSO

Search Guard also supports external authentication and Single Sign On (SSO) solutions. In most cases, these systems act as a proxy or store authentication information in HTTP header fields. Search Guard provides an HTTP proxy authenticator, which can read and interpret these fields. 

## Authorisation (authz)

After an authenticator has **verified** a user's **credentials**, an (optional) authorisation module can collect additional roles for the authenticated user from a configured backend. In most cases, this will be LDAP / Active Directory.

These roles are called **backend roles**.

## Users and roles

After the user is verified and roles have been retrieved, Search Guard will **map** the **user and any backend roles** to **Search Guard roles**. 

In some cases you want to map the backend roles 1:1 to Search Guard roles, but more often you want to map different backend roles and users to one Search Guard role.

## Permissions

Each interaction with Elasticsearch means that a particular **user** wants to **execute** an **action** on an Elasticsearch **cluster** and **one or more indices**. 

A permission defines:

* which role,
* can perform which action,
* against which cluster or index.

A definition of a permission that allows searching a particular index looks like:

* `indices:data/read/search*`

Permissions are defined per role and can be applied on a cluster or index level.

Search Guard ships with pre-defined groups of permissions like `READ`, `WRITE`, `SEARCH` etc.

## Action groups

An action group is an **alias for a set of permissions**. Action groups can be **nested**. 

For example, the following snippet shows two action groups, where the `SUGGEST` action group is referenced by the `SEARCH` action group:

```yaml
SEARCH:
  - "indices:data/read/search*"
  - "indices:data/read/msearch*"
  - SUGGEST
SUGGEST:
  - "indices:data/read/suggest*"
```

Action groups can be used in the role configuration instead of or in combination with fine-grained permissions like `indices:data/read/search*`.

Search Guard ships with a predefined set of useful action groups like `READ`, `WRITE`, `SEARCH` etc. We always keep the action groups up to date, so the **preferred way of configuring your roles is to use the pre-defined action groups**.

## The Search Guard index

All configuration settings for Search Guard, such as users, roles and permissions, are stored as documents in a special Search Guard index. This index is secured so that only an admin user with a special SSL certificate may write or read this index. You can define one or more of these certificates, called **admin certificates**, in elasticsearch.yml.

Keeping the configuration settings in an Elasticsearch index enables hot config reloading. This means that you can **change any of the user, role and permission settings at runtime, without restarting your nodes**. Configuration changes will **take effect immediately**. 

Also the authenticator configuration is hot reloadable, so you can add, remove or change atuthenticators at runtime as well.

You can load and change the settings from any machine which has access to your Elasticsearch cluster.  You do not need to keep any configuration files on the nodes themselves. 

The configuration consists of the following files. These are shipped with Search Guard as templates.

* sg\_config.yml - configure authenticators and authorisation backends.
* sg\_roles.yml - define roles and the associated permissions.
* sg\_roles\_mapping.yml - map backend roles, hosts and users to roles.
* sg\_internal\_users.yml - stores users, roles and hashed passwords (hash with hash.sh) in the internal user database.
* sg\_action\_groups.yml - define named permission groups.

Configuration settings are applied by pushing the content of one or more configuration files to the Search Guard secured cluster by using the `sgadmin` tool. For details, refer to the chapter [sgadmin](sgadmin.md). 