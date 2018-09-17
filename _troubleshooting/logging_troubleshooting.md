---
title: Search Guard logging
slug: troubleshooting-setting-log-level
category: troubleshooting
order: 50
layout: troubleshooting
---

<!--- Copryight 2017 floragunn GmbH -->

# Search Guard logging

For troubleshooting any problem with Search Guard, it is recommended to set the log level at least to `debug`.

Add the following lines in `config/log4j2.properties` and restart your node:

```
logger.searchguard.name = com.floragunn
logger.searchguard.level = debug
```

This already print out a lot if helpful information in your log file. If this information is not sufficient, you can also set the log level to `trace`.

## Filter logs by username

Search Guard adds the currently logged in user to the log4j *Thread Context Map*. This makes it possible to exclude/filter log messages for certain users.

This is especially useful when debugging permission issues with Kibana. You can filter out any log messages from the internal Kibana server that would otherwise just clutter the logfile. To filter log statements from the user *kibanaserver* and *logstash*, add a *ThreadContextMapFilter* to the respective appender in `log4j2.properties`:
 
``` 
appender.console.type = Console
appender.console.name = console
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = [%d{ISO8601}][%-5p][%-25c{1.}] %marker%m%n
appender.console.filter.user.type = ThreadContextMapFilter
appender.console.filter.user.onMatch=DENY
appender.console.filter.user.onMisMatch=NEUTRAL
appender.console.filter.user.operator=or
appender.console.filter.user.value1.type=KeyValuePair
appender.console.filter.user.value1.key=user
appender.console.filter.user.value1.value=kibanaserver
appender.console.filter.user.value1.type=KeyValuePair
appender.console.filter.user.value1.key=user
appender.console.filter.user.value1.value=logstash

```

## Add username to log statements

You can also use the log4j *Thread Context Map* to add the username to any log statement. For example, this makes it possible to filter the slow log by username. The username is accessible in the thread context map by key *user*. You can add it to any log pattern by using `%x{user}`: 

```
appender.rolling.layout.pattern = [%d{ISO8601}][%-5p][%-25c{1.}][%x{user}] %marker%.-10000m%n

appender.index_search_slowlog_rolling.layout.pattern = [%d{ISO8601}][%-5p][%-25c] [%x{user}] %marker%.-10000m%n
```