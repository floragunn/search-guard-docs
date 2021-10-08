---
title: Snapshot, Restore, Index Aliases
html_title: Snapshot Restore
slug: snapshot-restore
category: rolespermissions
order: 1000
layout: docs
edition: community
description: Control access to the snapshot and restore features of Elasticsearch by using Search Guard.
---
<!---
Copyright 2017 floragunn GmbH
-->
# Aliases, snapshots and restore
{: .no_toc}

{% include_relative _includes/toc.md %}

## Index alias handling

Before applying any security checks, Search Guard first resolves any alias to the concrete index name(s). Index aliases are thus transparent to Search Guard. The same is true for 

* Index wildcards, also with multiple index names
  * e.g. `https://localhost:9200/i*ex*,otherindex/_search` 
* Date math index names
  * e.g.  `https://localhost:9200/<logstash-{now/d}>/_search/_search`
* Filtered index aliases 

Index names are resolved to their concrete index name(s) in

* REST requests
* Transport requests
* Search Guard role definitions

In practice this means that you do not need to grant permissions on index aliases in addition to granting permission on the concrete index names. For example, if you have an index alias `myalias` pointing to an index `myindex`, you only need to configure permissions for `myindex`. These permissions apply regardless whether the user accesses the index via `myalias` or `myindex`.

In detail, for every request Search Guard

* resolves any index name(s) in the request to the concrete index name(s)
* resolves any index name(s) in the user's role definitions to the concrete index name(s)
* applies permission checks based on the concrete index name(s)

## Handling multiple filtered index aliases

Filtered index aliases can be used to filter documents from the underlying concrete index. However, **using filteres aliases is not a secure way to restrict access to certain documents**. In order to do that, please use the [Document Level Security](dlsfls_dls.md) feature of Search Guard.

Because of this potential security leak, Search Guard detects and treats multiple filtered index aliases in a special way: You can either disallow them completely, or issue a warning message on `WARN` or `DEBUG` level.

The following entry in sg_config can be used to configure this:

```yaml
 searchguard:
    dynamic:		    
      filtered_alias_mode: <warn|nowarn|disallow>
```

| Name  | Description  |
|---|---|
| disallow | forbids multiple filtered index aliases completely |
| warn | default, logs a warning message if multiple filtered index aliases are detected on `WARN` level |
| nowarn | logs a warning message if multiple filtered index aliases are detected on `DEBUG` level |      

## Snapshot and Restore

The Search Guard configuration index contains sensitive data like user, role and permission information. Since this index can be part of any snapshot the default   behavior of Search Guard is to only allow restore operations **if an admin certificate is used**.

For curl, you need to specify the admin certificate with it's complete certificate chain and the key like:

```bash
curl --insecure --cert chain.pem --key kirk.key.pem -XPOST '<host>:9200/_snapshot/my_backup/snapshot_1/_restore?pretty'
```

## Enabling snapshot and restore for regular users

If you want to allow restore operations also for regular users without an admin certificate, you need to enable this feature explicitly in `elasticsearch.yml`:

```
searchguard.enable_snapshot_restore_privilege: true
```

### Required permissions

A role definition in `roles.yml` which allows snapshot and restore operations on all indices looks like:

```yaml
sg_snapshot_restore:
  cluster:
    - cluster:admin/repository/put
    - cluster:admin/repository/get
    - cluster:admin/snapshot/status
    - cluster:admin/snapshot/get
    - cluster:admin/snapshot/create
    - cluster:admin/snapshot/restore
    - cluster:admin/snapshot/delete
  indices:
    '*':
      '*':
        - indices:data/write/index
        - indices:admin/create
```

### Restoring a snapshot

A snapshot can only be restored if it does not contain global state and it does not contain the 'searchguard' index. 

In order to restore indices from a snapshot that does contain global state, you need to exclude it when performing the restore. If your snapshot also contains the Search Guard index, list the indices to restore explicitely:

```json
POST /_snapshot/my_backup/snapshot_1/_restore
{
  "indices": "humanresources",  
  "include_global_state": false
}
```

It is recommended to exclude the Search Guard index from all snapshots globally.
