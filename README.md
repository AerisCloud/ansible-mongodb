MongoDB Role
============

Configuration
--------------

```ini
[mongodb:children]
mongodb-somedc-prod

[mongodb-somedc-prod:children]
mongodb1-somedc-prod

[mongodb1-somedc-prod]
mongodb1.somedc.prod ansible_ssh_host=172.16.0.210
mongodb2.somedc.prod ansible_ssh_host=172.16.0.211
mongodb3.somedc.prod ansible_ssh_host=172.16.0.212

[mongodb1-somedc-prod:vars]
# Where MongoDB data will be stored (default: /var/lib/mongodb)
mongodb_data_path = /data/mongodb

# Should we use smallfiles? (default: false)
# http://docs.mongodb.org/manual/reference/configuration-options/#storage.smallFiles
mongodb_smallfiles = false

# Enables the durability journal to ensure data files remain valid and recoverable.
# http://docs.mongodb.org/manual/reference/configuration-options/#storage.journal.enabled
mongodb_nojournal = false

# It **MUST** be the name of the inventory group containing all the servers of your replica set.
mongodb_replica_set_name = mongodb1-somedc-prod

# Specifies a maximum size in megabytes for the replication operation log.
# Once the mongod has created the oplog for the first time,
# changing this option will not affect the size of the oplog.
# You have to follow [this procedure](http://docs.mongodb.org/manual/tutorial/change-oplog-size/) to change it.
# http://docs.mongodb.org/manual/reference/configuration-options/#replication.oplogSizeMB
# http://docs.mongodb.org/manual/core/replica-set-oplog/#replica-set-oplog-sizing
mongodb_oplog_size = 1024
```

#### Bootstrapping a Replica Set

To bootstrap a replica set, you need to add the `mongodb_bootstrap` variable with the name of of one of the members of the replica set.

Example:
```bash
ansible-playbook playbook.yml \
  --limit="mongodb" \
  --extra-vars="mongodb_bootstrap=node1.mongodb1.*"
```

*Notes:* 
* The value of `mongodb_bootstrap` must be the name of one of the replica set members.
* All the servers of the replica set must have `mongodb_replica_set_name` defined to the same value.

Replication
-----------

The minimum requirements for a replica set are: A primary, a secondary, and an arbiter.
Most deployments, however, will keep three members that store data: A primary and two secondary members.

### Arbiters

If you have an even number of servers, you need to define an [arbiter](http://docs.mongodb.org/manual/core/replica-set-arbiter/), otherwise the election won't happen.
Arbiters do not keep a copy of the data. However, arbiters play a role in the elections that select a primary if the current primary is unavailable.
Do not run an arbiter on systems that also host the primary or the secondary members of the replica set.

### Read Preference

All members of the replica set can accept read operations.
However, by default, an application directs its read operations to the primary member.
See [Read Preference](http://docs.mongodb.org/manual/core/read-preference/) for details on changing the default read behavior.

### Write concern

The [write concern](http://docs.mongodb.org/manual/core/write-concern/#write-concern-acknowledged) should be chosen depending of the need of your application.
It should be one of the following:
* [1](http://docs.mongodb.org/manual/core/write-concern/#acknowledged)
* [journaled](http://docs.mongodb.org/manual/core/write-concern/#journaled)
* [majority](http://docs.mongodb.org/manual/core/replica-set-write-concern/#modify-default-write-concern)

### Maintenance operations

* [Add an Arbiter to Replica Set](http://docs.mongodb.org/manual/tutorial/add-replica-set-arbiter/).
* [Add Members to a Replica Set](http://docs.mongodb.org/manual/tutorial/expand-replica-set/).
* [Remove Members from Replica Set](http://docs.mongodb.org/manual/tutorial/remove-replica-set-member/).
* [Replace a Replica Set Member](http://docs.mongodb.org/manual/tutorial/replace-replica-set-member/).

See also
---------

* [MongoDB manual](http://docs.mongodb.org/manual/)
* [MongoDB Replication Introduction](http://docs.mongodb.org/manual/core/replication-introduction/)
