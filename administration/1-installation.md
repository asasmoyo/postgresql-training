Installing PostgreSQL
=====================

# Installation

Follow installation instructions depending os OS at https://www.postgresql.org/download/

For our case, run these on both VMs:

``` bash
sudo sh -c 'echo "deb [arch=amd64] http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get --no-install-recommends install postgresql-13
```

Installing s3cmd to store WAL and Backups to storage:

``` bash
sudo apt install --no-install-recommends s3cmd
```

# Checking PostgreSQL cluster status

You can check cluster status using:

``` bash
pg_lscluster
```

Example output:

```
Ver Cluster Port Status Owner    Data directory              Log file
13  main    5432 down   postgres /var/lib/postgresql/13/main /var/log/postgresql/postgresql-13-main.log
```

`Data directory` is where the cluster data is located and `Log file` is where the cluster logs is located. PostgreSQL configuration files are located in `/etc/postgresql/13/main`.
