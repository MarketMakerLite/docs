---
description: >-
  A guide to installing TimescaleDB for financial data on an Ubuntu server.
  TimescaleDB official docs: https://docs.timescale.com/
---

# Installing TimescaleDB

{% hint style="info" %}
This guide was created using Ubuntu 20.04 LTS and PostgreSQL 14
{% endhint %}

## Fresh Install

If you want to install postgres by combining multiple drives into a logical volume (useful for cloud block storage, or large local database), please skip to the [#if-using-block-storage-aws-digital-ocean-etc](installing-timescaledb.md#if-using-block-storage-aws-digital-ocean-etc "mention") section. If you are installing locally, using a single drive, continue below:&#x20;

### Install PostgreSQL-14

Update apt, find latest version of Postgres and install

```bash
sudo apt update && sudo apt upgrade
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt -y update
sudo apt -y install postgresql-14
systemctl status postgresql
```

### Install TimescaleDB

```
sudo sh -c "echo 'deb [signed-by=/usr/share/keyrings/timescale.keyring] https://packagecloud.io/timescale/timescaledb/ubuntu/ $(lsb_release -c -s) main' > /etc/apt/sources.list.d/timescaledb.list"
wget --quiet -O - https://packagecloud.io/timescale/timescaledb/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/timescale.keyring
sudo apt-get update
sudo apt install timescaledb-2-postgresql-14

#Update Conf file to include Timescale parameters
sudo timescaledb-tune
#Restart postgres
sudo service postgresql restart
```

### Editing Configurations Files

```sql
sudo nano /etc/postgresql/14/main/postgresql.conf
sudo nano /etc/postgresql/14/main/pg_hba.conf
sudo service postgresql restart
```

### Adding users and passwords

```
#Change existing user password
sudo -u postgres psql
\password postgres

#Change default password
sudo passwd postgres

#Create new users
sudo -u postgres createuser -P -s -e new_username
```

### If using Block Storage (AWS / Digital Ocean / Etc)

Ignore this section if you don't want to combine multiple drives into one (useful for cloud block storage such as EBS).&#x20;

{% hint style="info" %}
WARNING: THIS METHOD CURRENTLY DOES NOT WORK - EXTEND POSTGRES OVER A NON-ROOT LOGICAL VOLUME AT YOUR OWN RISK
{% endhint %}

```
#create logical group:
sudo pvcreate /dev/nvme1n1
sudo vgcreate LVG /dev/nvme1n1
sudo pvcreate /dev/nvme2n1
sudo vgextend LVG /dev/nvme2n1
sudo vgs

#create logical volume
sudo lvcreate -n LVM -L 999.99G LVG
sudo lvs
sudo mkdir /dev/LVG/LVM /mnt
sudo mkfs -t ext4 /dev/LVG/LVM
sudo mount /dev/LVG/LVM /mnt

#automatically start LVM on reboot
sudo nano /etc/fstab
#add the following line:
/dev/LVG/LVM /mnt ext4 defaults,nofail 0 2

#restart
sudo shutdown -r

#install postgres
sudo apt update && sudo apt upgrade
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt -y update
sudo apt -y install postgresql-14

sudo nano /etc/postgresql/14/main/postgresql.conf
#Change this line:
data_directory = ‘/var/lib/postgresql/14/main’
#to this:
data_directory = ‘/mnt/main’

sudo service postgresql stop
sudo nano /etc/init.d/postgresql
# change these lines
PGDATA=/mnt/main
PGLOG=/mnt/main
sudo chown -R postgres:postgres /mnt
sudo cp -Ri /var/run/postgresql /mnt/main
sudo su - postgres -c "initdb -D /mnt/main"
sudo service postgresql start

sudo nano /etc/postgresql/14/main/pg_hba.conf
#Add these lines

sudo service postgresql restart

#install timescale
sudo sh -c "echo 'deb [signed-by=/usr/share/keyrings/timescale.keyring] https://packagecloud.io/timescale/timescaledb/ubuntu/ $(lsb_release -c -s) main' > /etc/apt/sources.list.d/timescaledb.list"
wget --quiet -O - https://packagecloud.io/timescale/timescaledb/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/timescale.keyring
sudo apt-get update
sudo apt install timescaledb-2-postgresql-14

#Update Conf file to include Timescale parameters
sudo timescaledb-tune
#Restart postgres
sudo service postgresql restart
```

## Installing to existing PostgreSQL instance

[https://docs.timescale.com/timescaledb/latest/how-to-guides/install-timescaledb/self-hosted/ubuntu/installation-apt-ubuntu/](https://docs.timescale.com/timescaledb/latest/how-to-guides/install-timescaledb/self-hosted/ubuntu/installation-apt-ubuntu/)

Add the TimescaleDB third party repository and install TimescaleDB. This command downloads any required dependencies from the PostgreSQL repository:

```bash
sudo sh -c "echo 'deb [signed-by=/usr/share/keyrings/timescale.keyring] https://packagecloud.io/timescale/timescaledb/ubuntu/ $(lsb_release -c -s) main' > /etc/apt/sources.list.d/timescaledb.list"
wget --quiet -O - https://packagecloud.io/timescale/timescaledb/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/timescale.keyring
sudo apt-get update

# Now install appropriate package for PG version
sudo apt install timescaledb-2-postgresql-14
```

There are a [variety of settings that can be configured](https://docs.timescale.com/timescaledb/latest/how-to-guides/configuration/) for your new database. At a minimum, you need to update your `postgresql.conf` file to include `shared_preload_libraries = 'timescaledb'`. The easiest way to get started is to run `timescaledb-tune`, which is installed by default when using `apt`:

```bash
sudo timescaledb-tune

# Restart PostgreSQL instance
sudo service postgresql restart
```

## Creating a Hypertable

Create a new hypertable with the same index and structure of an existing Postgres table, copy over the data, and set a compression policy:

```sql
CREATE TABLE new_table (LIKE old_table INCLUDING DEFAULTS INCLUDING CONSTRAINTS INCLUDING INDEXES);

-- Assuming 'time' is the time column for the dataset
SELECT create_hypertable('new_table', 'time');

-- If there's an error on 'create_hypertable' run the following two statements, otherwise skip this step:
-- CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;
-- SELECT create_hypertable('new_table', 'time');

-- Insert everything from old_table
INSERT INTO new_table SELECT * FROM old_table;

-- Add compression, segmented by symbol/ticker
ALTER TABLE new_table SET (
  timescaledb.compress,
  timescaledb.compress_segmentby = 'symbol'
);

-- Add automatic compression at a custom interval:
SELECT add_compression_policy('new_table', INTERVAL '7 days');

-- More information on compression here: https://docs.timescale.com/timescaledb/latest/how-to-guides/compression/
```

