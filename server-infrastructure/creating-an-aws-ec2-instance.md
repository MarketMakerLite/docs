---
description: >-
  This guide will walk through creating and provisioning an EC2 instance -
  useful for data collection, automated trading, or running various bots.
---

# Creating an AWS EC2 Instance

## Part 1: Creating an Instance

### 1. Log into the AWS EC2 Console

You can visit the EC2 Instance Console at the following link, replacing the desired region with your own: [https://console.aws.amazon.com/ec2/v2/home?region=us-east-2](https://console.aws.amazon.com/ec2/v2/home?region=us-east-2)

_Note: Friends don't let friends choose US-East-1_&#x20;

__![](<../.gitbook/assets/image (12).png>)

### 2. Click "Launch Instance" and select the Ubuntu 20.04 LTS AMI with x86 architecture

![](<../.gitbook/assets/image (16).png>)

### 3. Choose an Instance Type, select t2.micro to utilize the free tier

![](<../.gitbook/assets/image (15).png>)

### 4. Configure Instance&#x20;

Select a specific subnet - this will be the same subnet used for all EC2 services, allowing multiple instances and other services to interact with each other. The actual subnet chosen does not matter (see image below) but it does need to be consistent.&#x20;

![](<../.gitbook/assets/image (7).png>)![](<../.gitbook/assets/image (5).png>)

### 5. Select Storage Options

I prefer to use GP3 type SSD storage, the size of the volume is optional between 8gb - 16tb, the cost is around \~$10 per 100GB.&#x20;

![](<../.gitbook/assets/image (14).png>)

### 6. Create Security Group

Add at least the following IPs to your security group to ensure access. You can find your external IP address here: [https://www.whatismyip.com/](https://www.whatismyip.com/) or just select "My IP" as the source. ![](<../.gitbook/assets/image (9).png>)

| Type        | Protocol | Port Range | Source                     |
| ----------- | -------- | ---------- | -------------------------- |
| SSH         | TCP      | 22         | My IP                      |
| HTTP        | TCP      | 80         | My IP                      |
| All Traffic | All      | All        | AWS Default Security Group |
| All Traffic | All      | All        | My IP                      |
| PostgreSQL  | TCP      | 5432       | My IP                      |

### 7. Review Instance and Launch

Once reviewing that all settings are correct and you're ready to launch the instance, click the "Launch Instance" button.&#x20;

Next a pop up screen will ask you to create a new Key Pair (or use an existing one). Your key pair is similar to a password, and is the ONLY way to access your new EC2 instance. To read more about key pairs, see here: [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html).&#x20;

&#x20; ![](<../.gitbook/assets/image (19).png>)

From the screen above, choose RSA for the Key pair type, and type a name that is easy to remember/type. Save the .pem file in a secure and easy to access location on your computer.&#x20;

### 8. Launch Instance

![](<../.gitbook/assets/image (6).png>)

## Part 2: Setting Up Your New Instance

### 1. Create an Elastic IP (Optional)

#### A.) Select Elastic IPs from the left hand menu in the EC2 Console. An _Elastic IP address_ is a static IPv4 address designed for dynamic cloud computing.&#x20;

![](<../.gitbook/assets/image (3).png>)

#### B.) Click Allocate Instance

![](<../.gitbook/assets/image (10).png>)

You will now see an allocated static IPv4 address in your Elastic IP console

#### C.) Click Actions -> Associate&#x20;

Select your newly created EC2 instance and associate the Elastic IP

![](<../.gitbook/assets/image (1).png>)

Click "Associate" and you'll see that the Elastic IP is now attached to your EC2 instance. This allows us to always use the same IP address, regardless if the instance is stopped & restarted (which would generally refresh the internal IPv4 address).

#### D.) Write down or remember your Elastic IP address, this is how we will connect to the instance

### 2. Accessing your Instance via SSH

#### A.) Open up a terminal / command prompt window and CD to the location where you saved your key-pair.pem file

![](<../.gitbook/assets/image (18).png>)

Run the following command to SSH into your EC2 instance (replace the x's with your Elastic IP address)

```
ssh -i key-pair.pem ubuntu@XX.XX.XX.XX
```

If prompted, type "yes" to confirm that you would like to add the ECDSA key fingerprint to your list of known hosts. You will then see the Ubuntu Server "home" page.&#x20;

![](<../.gitbook/assets/image (17).png>)

### 3. General Provisioning

First we'll start by making sure everything is up to date by running the following command, restart to apply any changes and then login to your instance again (restarting takes approx. 30sec)

```
#Apply Updates
sudo apt update && sudo apt upgrade

#Restart Instance
sudo shutdown -r now

#Login Again
ssh -i key-pair.pem ubuntu@XX.XX.XX.XX

#Install Pip
sudo apt install python3-pip

#Install Virtual-environments
sudo apt install python3-venv
sudo pip3 install virtualenv

```

### 4. Copying local files to and from your instance

You can copy local folders / files from your personal computer to your new EC2 instance. This makes it easy to develop locally (with a GUI) and deploy your production code in the cloud.&#x20;

To copy a file from your local machine to your EC2 instance, use the following code (replace \~/Desktop with the location of your key-pair):&#x20;

```
scp -i ~/Desktop/key-pair.pem -r [folder_name] ubuntu@[Your.Elastic.IP]:~/[folder_name]
```

If you would like to copy (PUSH) from your EC2 instance to your local computer, from the SSH window of your EC2 instance, use the following code:&#x20;

```
scp -r [folder_name] localuser@[Local.IP.Address]:~/[folder_name]
```

Or to PULL from your EC2 instance to a local machine using your local terminal/command prompt window, use the following code (this will pull the requested file/folder to your current location in the terminal window):&#x20;

```
scp -i ~/Desktop/key-pair.pem ubuntu@[Your.Elastic.IP]:/home/ubuntu/Folder-Name
```

## Part 3: Optional Software / Settings

### Creating an Ubuntu Service (FastAPI used as an example)

```
#Create a new service file:
sudo nano /etc/systemd/system/fastapi.service

#Add the following to the new file:
[Unit]
Description=FastAPI
After=multi-user.target
[Service]
Type=simple
Restart=always
restartSec=5s
ExecStart=/home/ubuntu/folder-name/env/bin/uvicorn /home/ubuntu/folder-name/main:app --reload --port 8000 --host 0.0.0.0 >StandardOutput=append:/home/ubuntu/logs/fastapi.log
StandardError=append:/home/ubuntu/logs/fastapi.log
[Install]
WantedBy=multi-user.target

#Close Nano
^X
y
[Enter]

#Refresh services
sudo systemctl daemon-reload

#Enable service (auto-launch e.g. persist when instance is restarted)
sudo systemctl enable fastapi.service

#Start service
 sudo systemctl start fastapi.servicebasInstalling Caddy (Web Server / Reverse Proxy)
```

### Installing PostgreSQL

#### Download and install the latest version of Postgres

```bash
sudo apt update && sudo apt upgrade
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt -y update
sudo apt -y install postgresql-14
systemctl status postgresql
```

Edit the config files to match the following screenshots

```sql
sudo nano /etc/postgresql/14/main/postgresql.conf
```

### ![](<../.gitbook/assets/image (20).png>)

```
sudo nano /etc/postgresql/14/main/pg_hba.conf

#Change the config file to the following: 

# Database administrative login by Unix domain socket                                                               
local   all             postgres                                scram-sha-256                                                                                                                                                           
# TYPE  DATABASE        USER            ADDRESS                 METHOD                                                                                                                                                                  
# "local" is for Unix domain socket connections only                                                                
local   all             all                                     peer                                                
# IPv4 local connections:                                                                                           
host    all             all             127.0.0.1/32            scram-sha-256                                       
host    all             all             0.0.0.0/0               scram-sha-256                                       
# IPv6 local connections:                                                                                           
host    all             all             ::1/128                 scram-sha-256                                       
host    all             all             ::/0                    scram-sha-256
```

![](<../.gitbook/assets/image (11).png>)

#### Restart Postgres

```
sudo systemctl restart postgresql
```

#### Add Postgres User

```
sudo -u postgres createuser -P -s -e [username]
```

#### Create Database

```
sudo -u postgres createdb [database_name]
sudo -u postgres psql
grant all privileges on database [database_name] to [username];
```

Download the latest version of PgAdmin to access your database from a remote host (e.g. on your main Mac/Windows PC): [https://www.pgadmin.org/download/](https://www.pgadmin.org/download/)

### Installing Redis

```
# Install Redis
sudo apt install redis-server
sudo systemctl status redis-server

# Remote Access
sudo nano /etc/redis/redis.conf
bind 127.0.0.1 ::1      # Comment out this line
bind 0.08.0.0 ::0  # Add this line
# requirepass foobared  # Uncomment this line to set password

sudo systemctl restart redis-server

# Ensure port is listening
ss -an | grep 6379

# Open Firewall
sudo ufw allow proto tcp from 192.168.121.0/24 to any port 6379

# Shutting Down Redis
# Get PID 
/var/run/redis/redis-server.pid

# Kill Server
kill [PID# Returned from above]
```

### Installing Caddy (web server / reverse proxy)

Caddy is used for when you need to allow inbound access to your EC2 instance, for example if you are running an API that takes requests, or serving a website.&#x20;

#### Installing Caddy

```
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo tee /etc/apt/trusted.gpg.d/caddy-stable.asc
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

Enable Caddy Reverse Proxy

```
sudo caddy reverse-proxy --from [Your.Elastic.IP] --to localhost:9000
```

Edit Caddy Config Files

```
sudo nano /etc/caddy/Caddyfile
```

Edit the config file to match the following screenshot

![](<../.gitbook/assets/image (1) (1).png>)

#### Editing Security Group

You will also need to update your AWS EC2 security group preferences (see[#6.-create-security-group](creating-an-aws-ec2-instance.md#6.-create-security-group "mention")) to allow inbound connections by adding the following:

| Type        | Protocol | Port Range | Source    |
| ----------- | -------- | ---------- | --------- |
| All Traffic | TCP      | 80         | 0.0.0.0/0 |
| All Traffic | TCP      | 443        | 0.0.0.0/0 |

####
