---
layout: post
title: MySQL 5.7.5- More variables in replication performance_schema tables
date: 2014-09-30
comments: true
categories:
- mysql
tags:
- mysql
- performance_schema
---

At MySQL, replication usability is of utmost importance to us. Replication information has long been part of SHOW commands, [SHOW SLAVE STATUS](http://dev.mysql.com/doc/refman/5.7/en/show-slave-status.html) occupying a major chunk of it. The other sources of replication information being

* [SHOW MASTER STATUS](http://dev.mysql.com/doc/refman/5.7/en/show-master-status.html),
* [SHOW BINLOG EVENTS](http://dev.mysql.com/doc/refman/5.7/en/show-binlog-events.html),
* [SHOW RELAYLOG EVENTS](http://dev.mysql.com/doc/refman/5.7/en/show-relaylog-events.html),
* [SHOW VARIABLES](http://dev.mysql.com/doc/refman/5.7/en/show-variables.html),
* [SHOW STATUS](http://dev.mysql.com/doc/refman/5.7/en/show-status.html),
* [ERROR LOGS](http://dev.mysql.com/doc/refman/5.7/en/error-log.html) etc. 

As the replication module grows further, there is a lot more monitoring information, so much that the present interfaces seem too rigid to accommodate all the information we would like to present. So we need to organize them in a more structured manner. In MySQL-5.7.2, we [introduced replication performance\_schema (P_S) tables](http://shivjijha.com/mysql/2013/09/22/MySQL-5.7:-Introducing-the-Performance-Schema-tables-to-monitor-Replication/) providing an SQL interface to monitor replication configuration and status partitioned into different tables, each table grouping logically related information. You can read more about the contents of these tables from [official MySQL documentation](http://dev.mysql.com/doc/refman/5.7/en/performance-schema-replication-tables.html).

In [MySQL-5.7.5](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/news-5-7-5.html) we added some more status variables to these performance\_schema tables to enable monitoring the latest replication features viz. [multi-source replication](http://www.slideshare.net/shiv4289/my-sql-labs-multi-source-replication) in [labs](http://labs.mysql.com/). Multi-source allows a MySQL slave to replicate from multiple sources (masters) directly. Talking of multi-source, one needs the replication information per source. So we added global variables that would be useful to extend to per-source scope to the replication performance\_schema tables to help monitor multi-source replication. Note that these variables still work for the single sourced replication and can still be accessed as:

    Show status like 'Slave_running';
    Show status like 'Slave_retried_transactions';
    Show status like 'Slave_last_heartbeat';
    Show status like 'Slave_received_heartbeats';
    show status like 'Slave_heartbeat_period';

Note though that the status variables are now mostly useful in single-source mode ONLY. If more sources are added, the status variables still just apply to the first source. For other replication sources (masters), the only way to access these variables is to use the replication performance\_schema tables as named in the table below. Here is how the names of server variables map to the names in the replication performance\_schema tables:


<table style="border:1px solid black">
<tr>
<th style="border:1px solid black;text-align:center"> Variable name </th>
<th style="border:1px solid black;text-align:center"> P_S Table Name </th>
<th style="border:1px solid black;text-align:center"> P_S field name </th>
</tr>
<tr>
<td style="border:1px solid black;text-align:left"> &nbsp;SLAVE_HEARTBEAT_PERIOD </td>
<td style="border:1px solid black;text-align:left"> &nbsp;replication_connection_configuration </td>
<td style="border:1px solid black;text-align:left"> &nbsp;HEARTBEAT_INTERVAL </td>
</tr>
<tr>
<td style="border:1px solid black;text-align:left"> &nbsp;SLAVE_RECEIVED_HEARTBEATS </td>
<td style="border:1px solid black;text-align:left"> &nbsp;replication_connection_status </td>
<td style="border:1px solid black;text-align:left"> &nbsp;COUNT_RECEIVED_HEARTBEATS </td>
</tr>
<tr>
<td style="border:1px solid black;text-align:left"> &nbsp;SLAVE_LAST_HEARTBEAT </td>
<td style="border:1px solid black;text-align:left"> &nbsp;replication_connection_status </td>
<td style="border:1px solid black;text-align:left"> &nbsp;LAST_HEARTBEAT_TIMESTAMP </td>
</tr>
<tr>
<td style="border:1px solid black;text-align:left"> &nbsp;SLAVE_RETRIED_TRANSACTIONS </td>
<td style="border:1px solid black;text-align:left"> &nbsp;replication_execute_status </td>
<td style="border:1px solid black;text-align:left"> &nbsp;COUNT_TRANSACTIONS_RETRIES </td>
</tr>
</table>


The variable 'slave\_running' reports whether the slave is running or not. This can be found by inspecting the two replication components (receiver and applier) separately to see if the receiver module is running or not by executing

    SELECT SERVICE_STATE FROM performance_schema.replication_connection_status;

And execute module is running or not by executing

    SERVICE_STATE FROM performance_schema.replication_execute_status;

Please try out multi-source replication and our new monitoring interface in the form of replication performance_schema tables. As always, your feedback is very valuable to us.
