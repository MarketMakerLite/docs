---
description: >-
  A guide to installing PostgreSQL for financial data on an Ubuntu server.
  PostgreSQL official docs: https://www.postgresql.org/docs/14/index.html
---

# Installing PostgreSQL

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
