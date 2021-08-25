Streaming Replication
=====================

PostgreSQL allows a PostgreSQL node to have multiple replicas. The source node is usually called by primary / master node. Replicas would connect to primary node then live stream for WAL records for data changes. Those data changes would be applied locally as soon as possible.

Primary node allows execution of read and write queries, while replicas only allows to receive read replicas. As of PostgreSQL 13, PostgreSQL doesn't have native support for multi primary replication.

Setting up replication in PostgreSQL is pretty straightforward:

1. Primary node must archive its WALs

1. Create replication role, the role must have `replication` options enabled

1. Set relatively high enough value for `max_wal_senders` and `max_replication_slots`

Let's do demonstration. We'll use `node-1` as primary and `node-2` as replicas. The steps are:

1. Configure WAL archival on `node-1`

1. Create replication role on `node-1`

1. Configure WAL restore on `node-2`

1. Take backup of `node-1` from `node-2` using `pg_basebackup` with `--write-recovery-conf` flag

1. Start PostgreSQL instance on `node-2`

1. Replication should be working. Test it by creating some data on `node-1`, the data should also available on `node-2`.

Creating replication role is done in `psql` with:

``` bash
create role replicator with login replication password 'rahasia';
```

Taking backup is of `node-1` is done by:

``` bash
pg_basebackup --pgdata=/var/lib/postgresql/13/main --write-recovery-conf --user=replicator --host=IP_OF_NODE_1
```

Above command will take backup of `node-1` and configure streaming replication as well (the `---write-recovery-conf`).

We can check if streaming replication is working by checking if primary has `walsender` process and replica has `walreceiver` process.

Another way to check if the setup is working is by checking the content of `pg_stat_replication` in `node-1`.

``` sql
postgres=# select * from pg_stat_replication;
-[ RECORD 1 ]----+------------------------------
pid              | 1813
usesysid         | 24581
usename          | replicator
application_name | 13/main
client_addr      | 172.16.71.5
client_hostname  |
client_port      | 57820
backend_start    | 2021-08-24 15:04:15.617699+00
backend_xmin     |
state            | streaming
sent_lsn         | 0/12000148
write_lsn        | 0/12000148
flush_lsn        | 0/12000148
replay_lsn       | 0/12000148
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
reply_time       | 2021-08-24 15:08:59.088851+00
```

# Limit of streaming replication

Streaming replication works by primary sending data from WAL files in `pg_wal` folder to replicas. If there any replica asks primary for older data that it doesn't have in `pg_wal`, the setup would fail.

However this can be solved by setting WAL restore with `restore_command`. This must be set the command to fetch WAL from WAL storage.

If `restore_command` is set, now streaming replication works by:

1. Replicas live streaming WAL records from primary

1. If for any reason replicas are too far behind, they would fetch WAL records by fetching WAL files from WAL storage

1. Once replicas catch up with primary, replicas back to live straming WAL records from primary
