---
title: Demo Installer (Linux/Mac)
html_title: Demo Installer
slug: demo-installer
category: quickstart
order: 200
layout: docs
edition: community
description: Search Guard ships with a demo installer for quickly setting up a working configuration. Use it for PoCs or checking out our features. 
resources:
  - "search-guard-presentations#quickstart|Search Guard Quickstart and First Steps (presentation)"
  - "search-guard-presentations#architecture-request-flow|Architecture and Request Flow (presentation)"
  - "search-guard-presentations#configuration-basics|Configuration Basics (presentation)"

---

<!--- Copyright 2020 floragunn GmbH -->

# Demo Installer 
{: .no_toc}

{% include toc.md %}

To quickly set up a Search Guard secured OpenSearch or Elasticsearch cluster:

1. Install the Search Guard Plugin to OpenSearch or Elasticsearch
2. Execute the Search Guard demo installation script

The demo installation script will setup and configure Search Guard on an existing OpenSearch/Elasticsearch cluster. It also installs demo users and roles for OpenSearch/Elasticsearch, Dashboards/Kibana and Logstash. It uses self-signed TLS certificates and unsafe configuration options, so **do not use in production!**

To use the (optional) Search Guard Dashboards/Kibana plugin which adds security and configuration features to Dashboards/Kibana:

1. Install the Search Guard Dashboards/Kibana plugin to Dashboards/Kibana
2. Add the minimal Dashboards/Kibana configuration to `kibana.yml`

## Install Search Guard on OpenSearch or Elasticsearch

Search Guard can be installed like any other OpenSearch/Elasticsearch plugin by using the `elasticsearch-plugin` command. 

* Download the [Search Guard version](../_docs_versions/versions_versionmatrix.md) matching your OpenSearch/Elasticsearch version
* Change to the directory of your OpenSearch/Elasticsearch installation and type:

```bash
bin/elasticsearch-plugin install -b file:///path/to/search-guard-{{site.searchguard.esmajorversion}}-<version>.zip
```


## Execute the demo installation script

Search Guard ships with a demo installation script. The script will:

* Add demo TLS certificates in PEM format to the `config` directory of OpenSearch/Elasticsearch
* Add the required TLS configuration to the `openearch.yml`/`elasticsearch.yml` file.
* Add the auto-initialize option to `openearch.yml`/`elasticsearch.yml`. This option will initialize the Search Guard configuration index automatically if it does not exist.
* Generate a `sgadmin_demo.sh` script that you can use for applying configuration changes on the command line

Note that the script only works with vanilla OpenSearch/Elasticsearch installations. If you already made changes to ``elasticsearch.yml``, especially the cluster name and the host entries, you might need to adapt the generated configuration.

To execute the demo installation:

* ``cd`` into `<OpenSearch/Elasticsearch directory>/plugins/search-guard-{{site.searchguard.esmajorversion}}/tools`
* Execute ``./install_demo_configuration.sh`` (``chmod`` the script first if necessary.)

The demo installer will ask if you would like to install the demo certificates, if the Search Guard configuaration should be automatically initialized and if cluster mode should be enabled. Answer as follows:

```bash
Search Guard {{site.searchguard.esmajorversion}} Demo Installer
 ** Warning: Do not use on production or publicly reachable systems **
Install demo certificates? [y/N] y
Initialize Search Guard? [y/N] y
Enable cluster mode? [y/N] n
```

* Install demo certificates
  * Whether to install the self-signed demo TLS certificates or not
* Initialize Search Guard
  * Whether to auto-initialize Search Guard with the demo configuration
  * If answered with `y`, Search Guard will initialize the configuration index with the files from the `<OpenSearch/Elasticsearch directory>/plugins/search-guard-{{site.searchguard.esmajorversion}}/sgconfig` directory if the index does not exist 
* Enable cluster mode
  * If answered with `y`, the `network.host` parameter will be set to `0.0.0.0` to bind to all interfaces
  * Depending on your system you may need to adjust the `vm.max_map_count` for OpenSearch/Elasticsearch to start
  * see [https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html) 

## Testing the installation

* Open ``https://localhost:9200/_searchguard/authinfo``.
* Accept the self-signed demo TLS certificate.
* In the HTTP Basic Authentication dialogue, use ``admin`` as username and ``admin`` as password.
* This will print out information about the user ``admin`` in JSON format.

## Applying configuration changes

The Search Guard configuration, like users, roles and permissions, is stored in a dedicated index in OpenSearch/Elasticsearch itself, the so-called Search Guard Index. 

Changes to the Search Guard configuration must be applied to this index by either

* Using the Search Guard Configuration GUI (Enterprise feature)
* Using the sgadmin command line tool with the generated admin certificate

For using the Search Guard Configuration GUI you need to install the Search Guard Dashboards/Kibana Plugin, as described below. 

If you want to use the sgadmin tool:

* Apply your changes to the demo configuration files located in `<OpenSearch/Elasticsearch directory>/plugins/search-guard-{{site.searchguard.esmajorversion}}/sgconfig`
* Execute the pre-configured sgadmin call by executing `<OpenSearch/Elasticsearch directory>/plugins/search-guard-{{site.searchguard.esmajorversion}}/tools/sgadmin_demo.sh`

This will read the contents of the configuration files in `<OpenSearch/Elasticsearch directory>/plugins/search-guard-{{site.searchguard.esmajorversion}}/sgconfig` and upload the contents to the Search Guard index. 

The sgadmin tool is very powerful and offers a lot of features to manage any Search Guard installation. For more information about sgadmin, head over to the [Using sgadmin](../_docs_configuration_changes/configuration_sgadmin.md) chapter.

## Install Search Guard on OpenSearch Dashboards or Kibana

The Search Guard Dashboards/Kibana plugin adds authentication, multi tenancy and the Search Guard configuration GUI to Open Search Dashboards and Kibana.

### OpenSearch Dashboards Installation

If you are using OpenSearch Dashboards, perform the following steps:

* Download the [Search Guard Dashboards plugin zip](../_docs_versions/versions_versionmatrix.md) matching your exact OpenSearch Dashboards version
* Stop OpenSearch Dashboards
* cd into your OpenSearch Dashboards installation directory
* Execute: `bin/opensearch-dashboards-plugin install file:///path/to/opensearch-dashboards-plugin.zip

If you've used the demo configuration to initializing Search Guard as outlined above, add the following lines to your `opensearch_bashboards.yml` and restart Dashboards:

```yaml
# Use HTTPS instead of HTTP
opensearch.hosts: "https://localhost:9200"

# Configure the Dashboards/Kibana internal server user
opensearch.username: "kibanaserver"
opensearch.password: "kibanaserver"

# Disable SSL verification because we use self-signed demo certificates
opensearch.ssl.verificationMode: none

# Whitelist the Search Guard Multi Tenancy Header
opensearch.requestHeadersWhitelist: [ "Authorization", "sgtenant" ]
```

### Kibana Installation

If you are using Kibana, perform the following steps:

* Download the [Search Guard Kibana plugin zip](../_docs_versions/versions_versionmatrix.md) matching your exact Kibana version
* Stop Kibana
* cd into your Kibana installation directory
* Execute: `bin/kibana-plugin install file:///path/to/kibana-plugin.zip

If you've used the demo configuration to initializing Search Guard as outlined above, add the following lines to your `kibana.yml` and restart Kibana:

```yaml
# Use HTTPS instead of HTTP
elasticsearch.hosts: "https://localhost:9200"

# Configure the Dashboards/Kibana internal server user
elasticsearch.username: "kibanaserver"
elasticsearch.password: "kibanaserver"

# Disable SSL verification because we use self-signed demo certificates
elasticsearch.ssl.verificationMode: none

# Whitelist the Search Guard Multi Tenancy Header
elasticsearch.requestHeadersWhitelist: [ "Authorization", "sgtenant" ]

# X-Pack security needs to be disabled for Search Guard to work properly
xpack.security.enabled: false
```

## Start Dashboards/Kibana

After Dashboards/Kibana is started, it will begin optimizing and caching browser bundles. This process may take a few minutes and cannot be skipped. After the plugin is installed and optimized, Dashboards/Kibana will continue to start.

## Testing the installation

* Open `http://localhost:5601/`.
* You should be redirected to the login page
* On the login dialogue, use `admin` as user name and `admin` as password.

If everything is set up correctly, you should see three new navigation entries on the left pane:

* Search Guard - the [Search Guard configuration GUI](../_docs_configuration_changes/configuration_config_gui.md)
* Tenants - to select a tenant for [Dashboards/Kibana Multitenancy](../_docs_kibana/kibana_multitenancy.md)
* Logout - to end your current session

## Applying configuration changes

The Search Guard configuration GUI allows you to edit

* Search Guard Roles - define access permissions to indices and types
* Action Groups - define groups of access permissions
* Role Mappings - Assign users by username or their backend roles to Search Guard roles
* Internal User Database - An authentication backend that stores users directly in OpenSearch/Elasticsearch

Furthermore you can view your currently active license, upload a new license if it has expired, and display the Search Guard system status.

## Where to go next

If you have not already done so, make yourself familiar with the [Search Guard Main Concepts](../_docs_quickstart/main_concepts.md). 

After that, configure roles and access permissions by either modifying the configuration files and uploading them via `sgadmin`, or use the Dashboards/Kibana configuration GUI to change them directly. 

* [Using and defining action groups](../_docs_roles_permissions/configuration_action_groups.md)
* [Defining roles and permissions](../_docs_roles_permissions/configuration_roles_permissions.md)
* [Mapping users to Search Guard roles](../_docs_roles_permissions/configuration_roles_mapping.md)
* [Adding users to the internal user database](../_docs_roles_permissions/configuration_internalusers.md)

If you want to use more sophisticated authentication methods like Active Directory, LDAP, Kerberos or JWT, [configure your existing authentication and authorisation backends](../_docs_auth_auth/auth_auth_configuration.md) in `sg_config.yml`.

For fine-grained access control on document- and field level, use the Search Guard [Document and field level security module](../_docs_dls_fls/dlsfls_dls.md).

If you need to stay compliant with security regulations like GDPR, HIPAA, PCI, ISO or SOX, use the [Search Guard Audit Logging](../_docs_audit_logging/auditlogging.md) to generate and store audit trails.

And if you need to support multiple tenants in Dashboards/Kibana, use [Dashboards/Kibana Multitenancy](../_docs_kibana/kibana_multitenancy.md) to separate Visualizations and Dashboards by tenant.