Point in Time Recovery
======================

Point in time recovery allows us to recover PostgreSQL state at certain point of time. This is useful to recover data when we accidentally delete them.

This requires:

1. Backup taken by `pg_basebackup` before the time to recover

2. WALs taken since backup taken until the time to recover

Let's demonstrate this. The scenario will be as follows:

1. Configure WAL archival on `node-1` like in chapter 5

1. Create backup of `node-1`, then restore it on `node-2` but don't start it yet

1. Create some data on `node-1`, print current timestamp

1. Delete the data on `node-1`, then force PostgreSQL to upload WAL by `select pg_switch_wal()`

1. Configure PITR on `node-2` to recover at timestamp taken after data was created on `node-1`, then start the PostgreSQL instance.

1. Check that the data are present on `node-2`.

Configuring PITR on `node-2` is done by creating `/etc/postgresql/13/main/conf.d/pitr.conf`

``` bash
restore_command = 's3cmd --access_key="" --secret_key="" --host=minio.203.6.148.201.nip.io --host-bucket=minio.203.6.148.201.nip.io --no-ssl get s3://postgresql-training/wal/me/%f %p'
recovery_target_time = ''
recovery_target_inclusive = false
```

Set `recovery_target_time` to the timestamp after data was created on `node-1`. For example `2021-08-24 14:23:40.988834+00`.
