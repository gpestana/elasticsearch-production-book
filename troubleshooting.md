# 2. Data

## How to recover a red state cluster?

> **TL;DR**
> If data was permanently lost, first inspect which shards missing and which indexes are affected. After you know the problematic index, delete it. If you have backups of the damaged indexes, restore them after deleted.


A red state Elasticsearch cluster happens when data that once was stored in the cluster is missing and cannot be recovered. If the primary shard and its replicas are missing in the cluster, the index that were part of the missing primary shard turns red. The Elasticsearch cluster turns red when at least one index is red too.

The first thing to do when the cluster turns red is to understand why did that happen. One possible cause is that more than one data node are down - the nodes where the primary shard and its replicas are stored. If that is the case, then a solution is to bring the nodes back up and wait for them to join the cluster again.

Another possible scenario is when one or more shards and its replicas currupted for some reason. It may have been that a couple of nodes' volumes were completly wiped out, for example. In this case, one way to bring the cluster again to a green state is by deleting the red indexes and restore them to the latest backup snapshot.

The practical steps for restoring the status of the cluster when shards cannot be recovered automatically are:

#### 1. Check which indexes are affected

First and foremost, we need to know which indexes are causing the cluster to be red:

```
  curl localhost:9200/_cluster/health?level=indices
```

```
{
   "cluster_name": "elasticsearch_cluster01",
   "status": "red",
   "timed_out": false,
   "number_of_nodes": 7,
   "number_of_data_nodes": 7,
   "active_primary_shards": 90,
   "active_shards": 180,
   "relocating_shards": 0,
   "initializing_shards": 0,
   "unassigned_shards": 20
   "indices": {
      "index_1": {
         "status": "green",
         "number_of_shards": 5,
         "number_of_replicas": 1,
         "active_primary_shards": 5,
         "active_shards": 10,
         "relocating_shards": 0,
         "initializing_shards": 0,
         "unassigned_shards": 0
      },
      "index_2": {
         "status": "red", 
         "number_of_shards": 5,
         "number_of_replicas": 1,
         "active_primary_shards": 0,
         "active_shards": 0,
         "relocating_shards": 0,
         "initializing_shards": 0,
         "unassigned_shards": 10 
      },
     ....
 }
```

*index_2* has 10 unassigned shards, which are causing the cluster to be in red state. 


#### 2. Delete red indexes

Since it is not possible to restore backup data when the cluster is in red state, first the faulty indexes must be deleted. Run the following API request for each of the red indexes:

```
  curl -XDELETE 'http://localhost:9200/<name_of_red_index>'

```

#### 3. Restore index from backups

To restore a given index from snapshots, make th folowing REST request:

```
  curl -XPOST http://localhost:9200/_snapshot/<backup_name>/snapshot_1/_restore -d '
	{
    	"indices": "index_1", 
	}'
```


Once the indexes are restored, the cluster should recover completely and turn to green state again.


## How to remove stuck pending tasks

> TLDR: To remove pending tasks force a master node re-election or soft reboot the cluster.

Pending tasks are queued tasks that the master node needs to perform. Usually, the master node is not the cluster's bottleneck and it can process the actions as fast as required. In rare cases, the master node may start misbehaving and pile up tasks. This leads to performance issues or cluster misbehaving.

There is no explicit way to remove pending tasks. if the master node start piling up pending task often, you need to understand the root causes and fix it. Excess of pending tasks lead to overall performance degradation. The most probable causes for it might be


- **The cluster state is very large and is updated frequently**: The load in the master node can become too heavy if you have several indexes and each index has dynamic fields. In this case, you either refactor the data model or add more resources to the master node. Another way to solve the issue is to scale horizontally by starting a new cluster in order to distribute the load among multiple master nodes.

- **All data nodes are unreachable**: This is obviously a problem and will turn the cluster into red state. Once at least one data node is reacheable again, the master will restart to perform the tasks.

- **The master node instance is unresponsive**: When the master node instance misbehaves and is not able to perform tasks, the master node might get clogged up. In those cases, the solution is to force a master re-election and fix the underlying problem in the master node.


In the case that the cluster has pending tasks that are affecting other tasks, to restart the master node will solve the problem. In the following example:

```
$ curl 'localhost:9200/_cat/pending_tasks?v'
```

```
insertOrder timeInQueue priority source
      21611        6.7h NORMAL   put_repository [backup_index_0123]
      28238       15.8s URGENT   create-index-template [logstash-event], cause [api]
      28239        8.7s URGENT   create-index-template [logstash-metric], cause [api]
      28241        2.3s URGENT   create-index-template [logstash-metric], cause [api]
      28231       18.6s HIGH     put-mapping [index_ABC]
      28230       27.9s HIGH     put-mapping [index_ABC]
     ...
```

The ``put_repository`` task got stuck and several other tasks started to pile up. The solution in mishaving cases is to force a master re-election by restarting the Elasticsearch service in the master node:

``# service elasticsearch restart``
