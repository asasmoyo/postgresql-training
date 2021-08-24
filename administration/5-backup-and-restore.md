Backup and Restore
==================

# Creating backup

PostgreSQL has tool named `pg_basebackup` that can take backup from a PostgreSQL Cluster. The usage is straightforward:

```
pg_basebackup --pgdata=BACKUP_DIR_DEST --host=PG_HOST --username=PG_USER --password
```

If you want to create backup user, please keep in mind to grant `replication` to the user. `pg_basebackup` requires user to have `replication` option.

Let's take backup on `node-1`. Change into `postgres` user with:

``` bash
sudo su postgres
```

Then take backup of the PostgreSQL instance in `node-1`:

``` bash
pg_basebackup --pgdata=backup --format=tar
```

This will create:

``` bash
postgres@node-1:~$ tree backup/
backup/
├── backup_manifest
├── base.tar
└── pg_wal.tar
```

Now you can send the backup to backup storage. For example if you are using s3 / minio:

``` bash
s3cmd --access_key='' --secret_key='' --host=minio.203.6.148.201.nip.io --host-bucket=minio.203.6.148.201.nip.io --no-ssl put backup/ s3://postgresql-training/backup/me/backup1/ --recursive
```

# Restoring backup

Download the backup on `node-2` into its data directory:

``` bash
sudo pg_ctlcluster 13 main stop
sudo rm -rf /var/lib/postgresql/13/main
sudo mkdir /var/lib/postgresql/13/main
sudo s3cmd --access_key='' --secret_key='' --host=minio.203.6.148.201.nip.io --host-bucket=minio.203.6.148.201.nip.io --no-ssl get s3://postgresql-training/backup/me/backup1/ /var/lib/postgresql/13/main/ --recursive
```

Then start PostgreSQL instance:

```
sudo chown -R postgres:postgres /var/lib/postgresql/13/main
sudo chmod -R 700 /var/lib/postgresql/13/main
sudo pg_ctlcluster 13 main start
```

PostgreSQL by default requires data directory to be owned by `postgres:postgres` and to have `700` permission.

PostgreSQL configurations in `/etc/postgresql/13/main` must be backed up separately.

# Archiving Write Ahead Log (WAL)

> Write-Ahead Logging (WAL) is a standard method for ensuring data integrity. A detailed description can be found in most (if not all) books about transaction processing. Briefly, WAL's central concept is that changes to data files (where tables and indexes reside) must be written only after those changes have been logged, that is, after log records describing the changes have been flushed to permanent storage. If we follow this procedure, we do not need to flush data pages to disk on every transaction commit, because we know that in the event of a crash we will be able to recover the database using the log: any changes that have not been applied to the data pages can be redone from the log records. (This is roll-forward recovery, also known as REDO.)

> https://www.postgresql.org/docs/13/wal-intro.html

We need to configure `archive_command` to archive WAL. Create `/etc/postgresql/13/main/conf.d/wal_archival.conf`:

``` bash
archive_mode = on
archive_command = "s3cmd --access_key='' --secret_key='' --host=minio.203.6.148.201.nip.io --host-bucket=minio.203.6.148.201.nip.io --no-ssl put %p s3://postgresql-training/wal/me/%f"
wal_level = 'replica'
```

# Storing backup

Backup should be stored in safe storage, preferably in different datacenter. This helps keeping the backup safe when the main datacenter is completely destroyed.

Few most popular options are Google Cloud Storage and AWS S3. They even have support for storing files with redundancy, the files should still be safe when one or maybe more datacenter break.

# Why do we archive WALs if we have backup?

They both are important to have to have robust backup pipeline. With backups, we can restore database to at specific state when the backup was taken. In order to have least data loss in case of PostgreSQL instance completely gone, we would have to create backup as frequently as possible. However it is not practical, few reasons are: it is expensive to store backups especially when the database size is large and taking backup could affect performance of the database instance.

However, as mentioned before that WAL basically contains logs of changes. If we have a backup taken at certain time and WALs from that time until now, we could apply those WALs to the backup then we would get the current state of database. Typical backup pipeline is done by taking backups for every days / week and store WALs for the past 1-2 months.
