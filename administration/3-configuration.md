PostgreSQL Configuration
========================

Configurations are located in folder `/etc/postgresql/13/main`. Notice the same `13/main` shared between configuration directory and data directory. `13/main` is the cluster name, it consists of version and the cluser folder name. Single machine could host multiple PostgreSQL versions at the same time and single version can have multiple clusters running at the same time.

Important files at the configuration folder:

1. `pg_hba.conf` containes PostgreSQL internal firewall. This configures who can access which database from specified address with specified authentication method.

2. `postgresql.conf` contains PostgreSQL configuration parameters. It includes all files with name ending with `.conf` under `conf.d` folder, therefore configurations can be splitted into multiple files.

## Tuning configuration

Check https://pgtune.leopard.in.ua/#/
