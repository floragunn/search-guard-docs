---
title: Document-level security
html_title: Document-level security
permalink: document-level-security
category: dlsfls
order: 100
layout: docs
edition: enterprise
description: Use Document- and Field-Level Security to implement fine grained access control to documents and fields in your Elasticsearch cluster.
resources:
  - "search-guard-presentations#dls-fls|Document- and Field-level security (presentation)"
  - https://search-guard.com/document-field-level-security-search-guard/|Document- and field-level security with Search Guard (blog post)
  - https://search-guard.com/attribute-based-document-access/|Attribute based document access (blog post)

---
<!---
Copyright 2020 floragunn GmbH
-->

# Document-level security
{: .no_toc}

{% include toc.md %}

Document-level security restricts a user's access to certain documents within an index. To enable document-level security you configure an Elasticsearch query that defines which documents are accessible and which not. Only documents matching this query will be visible for the role that the DLS is defined for.

The query supports the full range of the Elasticsearch query DSL, and you can also use user attributes to make the query dynamic. This is a powerful feature to implement access permissions to documents based on user attributes stored in Active Directory / LDAP or a JSON web token.

## Example

Let's imagine we have an index called `humanresources`. This index contains documents with type `employees`, and these documents have a field called `department`. We want to define a query that allows access to all employee documents, except those where the department is set to "Management". 

The respective query to filter these documents in regular query DSL would look like:

```json
{
  "query": {
    "bool": {
      "must_not": { "match": { "department": "Management" }}
    }
  }
}
```

You can use this query to define the DLS in `sg_roles.yml`:

```yaml
hr_employee:
  index_permissions:
    - index_patterns:
      - 'humanresources'
      allowed_actions:
        - ...
      dls: '{ "bool": { "must_not": { "match": { "department": "Management" }}}}'
```

If a user has the role `hr_employee`, Search Guard now filters all documents where the `department` field is set to "Management" from any search result before passing it back to the user.

The format of the query is the same as if it was used in a search request. The specified query expects the same format as if it was defined in the search request and supports ELasticsearch’s full Query DSL.

This means that you can make the DSL query as complex as you want, but since it has to be executed for each query, this, of course, comes with a small performance penalty.

## Dynamic queries: Variable substitution

Search Guard supports variables in the DLS query. With this feature, you can write dynamic queries based on the current users's attributes. 

## Username

You can use the variable `${user.name}` in the DLS query, and Search Guard will replace it with the username of the currently logged in user.

Let's imagine that each employee document has a field called `manager`, which contains the username of the employee's manager. Each logged in user should only have access to employees he manages. You can do so by defining:

```yaml
hr_employee:
  index_permissions:
    - index_patterns:
      - 'humanresources'
      allowed_actions:
        - ...
      dls: '{"term" : {"manager" : "${user.name}"}}'
```

Before the DLS query is applied to the result set, `${user.name}` is replaced by the currently logged in user. You can use this variable repeatedly in the DLS query if required.

## User Roles

You can use the variable `${user.roles}` in the DLS query, and Search Guard will replace it with a comma-delimited list of the backend roles of the current user.

Let's imagine that each employee document has an array field called `role`, which contains backend role names and at least one of them is needed to access this document. You can do so by defining:

```yaml
hr_employee:
  index_permissions:
    - index_patterns:
      - 'humanresources'
      allowed_actions:
        - ...
      dls: '{"terms" : { "role" : [${user.roles}]}}'
```

Before the DLS query is applied to the result set, `${user.roles}` is replaced with a comma-delimited list of the backend roles of the current user. You can use this variable repeatedly in the DLS query if required.

## LDAP and JWT user attributes

Any authentication and authorization backend can add additional user attributes that you can then use for variable substitution.

For JWT, these are the claims from your JWT token. For Active Directory and LDAP these are all attributes stored in the user's Active Directory / LDAP record.  For JWT, the attributes start with `attr.jwt.*`, for LDAP they start with `attr.ldap.*`. 

If you're unsure, what attributes are accessible you can always access the `/_searchguard/authinfo` endpoint to check. The endpoint will list all attribute names for the currently logged in user.

### JWT Example

If the JWT contains a claim `department`:

```json
{
  "name": "John Doe",
  "roles": "admin, devops",
  "department": "operations"
}
```

You can use it like:

```yaml
hr_employee:
  index_permissions:
    - index_patterns:
      - 'humanresources'
      allowed_actions:
        - ...
      dls: '{"term" : {"department" : "${attr.jwt.department}"}}'
```

The DLS query in this case will only return documents where the `department` field equals the `department` claim in the users JWT. In this case, it only returns documents where the `department` field equals `operations`.

You can also combine multiple variables and username substitution in the same DLS query.

### Active Directory / LDAP Example

If the Active Directory / LDAP entry of the current user contains an attribute `department`, you can use it in a DLS query like:

```yaml
hr_employee:
  index_permissions:
    - index_patterns:
      - 'humanresources'
      allowed_actions:
        - ...
      dls: '{"term" : {"department" : "${attr.ldap.department}"}}'
```
 
## Multiple roles and document-level security

A user can be member of more than one role, and each role can potentially define a different DLS query for the same index. In this case, all DLS queries are collected and combined with `OR`.

If a user has a role that defines DLS restrictions on an index, and another role that does not place any DLS restrictions on the same index, the restrictions defined in the first role still apply.

You can change that behaviour so that a role that places no restrictions on an index removes any restrictions from other roles. This can be enabled in `elasticsearch.yml`: 

```
searchguard.dfm_empty_overrides_all: true
```


## Performance considerations

A DLS query can be as simple or complex as necessary, and you can use the full range of Elasticsearch's query DSL. Regarding the performance overhead, think of the DLS query as an additional query executed on top of your original one. 

## DLS/FLS Execution Order

If you use both DLS and FLS, all fields that you are basing the DLS query on must be visible, i.e. not filtered by FLS. Otherwise, your DLS query will not work properly. 