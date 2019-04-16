---
title: Manage data consistency and entropy
description: Maintain data consistency in InfluxDB Enterprise clusters with manual repairs, replacing missing shards, or using the Anti-Entropy service
menu:
  enterprise_influxdb_1_7:
    menu: Manage data consistency
    weight: 45
    parent: Replication
---

## Overview

InfluxDB Enterprise clusters use data replication to ensure fault tolerance and reliability.

Horizontal scaling adds capacity by increasing the number of nodes to a system so that each node can share the processing.

Unexpected events, though, can cause interruptions in the normally smooth and reliable replication of shards.

When data 

When data is written to InfluxDB Enterprise clusters and replicated in multiple shards (copies of data),  across data nodes in the cluster. The number of shard copies is determined by the replication factor (RF) and during normal operations, data is written to a shard and replicated to the shard copies without isssues. At any instant of time, the multiple copies are not exact copies of each other, but as the data is replicated among the shards, the data is eventually consistent and are replicas of each other. Due to unanticipated events, however, the copies of data between the shards can become inconsistent, and the differences between the copies need to be reconciled.

## Indicators of data inconsistency

When data becomes inconsistent between two or more shards, you might notice the following:

### Inconsistent query results

When you see significantly inconsistent query results for a query that covers the same time ranges, the query might be executing against different data nodes. If the shards between two nodes are inconsistent, then the results can different significantly. 

To determine if two shards are inconsistent, you can run a query on data nodes with shard replicas to get the count of query results and compare the counts. Small differences can be due to the eventually consistency of the shards, but significant differences can be due to data inconsistencies.

#### Example

When running queries against an InfluxDB Enterprise cluster, each query can be routed to different data nodes.
If entropy affects data within the queried range, the same query will return different results depending on which data node it is run against.

##### Initial query

```sql
SELECT mean("usage_idle") WHERE time > '2018-06-06T18:00:00Z' AND time < '2018-06-06T18:15:00Z' GROUP BY time(3m) FILL(0)

name: cpu
time                  mean
----                  ----
1528308000000000000   99.11867392974537
1528308180000000000   99.15410822137049
1528308360000000000   99.14927494363032
1528308540000000000   99.1980535465783
1528308720000000000   99.18584290492262
```

##### Repeated query

In this example, the same query is run again, but the results are different for three rows.
This indicates that data is missing in the queried time range.

```sql
SELECT mean("usage_idle") WHERE time > '2018-06-06T18:00:00Z' AND time < '2018-06-06T18:15:00Z' GROUP BY time(3m) FILL(0)

name: cpu
time                  mean
----                  ----
1528308000000000000   99.11867392974537
1528308180000000000   0
1528308360000000000   0
1528308540000000000   0
1528308720000000000   99.18584290492262
```

### Flapping metrics

When metrics you are mon"flapping" — rapidly changing between values — 

- Flapping metrics
  - Rapid alert status changes.
    - Alerts that fire and clear repeatedly in a short time period.
    - Values in graphs change erratically or switch rapidly.
- Shard copies are not the same size.

## Causes of data inconsistency

Data inconsistency between shards, or entropy, can occur when unanticipated events occur.
Some of the most frequent causes of entropy in InfluxDB Enterprise clusters include:

### Hardware and server failures

Server crashes or power interruptions can cause data inconsistencies as a side effect. 

Data node panics can cause inflight data to be dropped. 
Inflight data can be dropped when a node crashes after the write was replicated, but before the `fsync` operation on the remote node,

### Dropping or deleting measurements or series

Dropping or deleting series or measurements can result in data inconsistencies.

### Hinted Handoff overflows

During normal operations, the [Hinted Handoff (HH)](/enterprise_influxdb/v1.7/concepts/clustering/#hinted-handoff) queue allows data nodes to temporarily cache writes destined for another data node when the target data node is unreachable. If a data node crashes or is offline for too long, the Hinted Hangoff queue can overflow, resulting in data loss and data inconsistencies between shards in a shard group.

[Hinted Handoff configuration settings](/enterprise_influxdb/v1.7/administration/config-data-nodes/#hinted-handoff-settings) on data nodes, including `max-size` and `max-age`, might affect whether an overflow occurs.

### Network partitions

A network partition occurs when the InfluxDB cluster nodes cannot talk to each other.
Depending on tradeoffs between consistency and availability, 

#### `max-age`


#### `max-size = "

, If a data node is down, or offline, for longer than 
overflows
- Batch write interruptions resulting in partial data writes

## Monitor data inconsistency and Anti-Entropy service

## Repair data inconsistencies and entropy

### Repair data inconsistencies manually

### Repair data inconsistencies using the Anti-Entropy service

Overwriting everything has two consequences:

- Recompaction to read-optimize including the added missing data
- Slower queries temporarily until recompaction finished


