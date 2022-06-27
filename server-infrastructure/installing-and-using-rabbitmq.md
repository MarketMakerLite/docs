# Installing and Using RabbitMQ

{% hint style="info" %}
Discontinued support in favor of Redis
{% endhint %}

```
wget -O- https://packages.erlang-solutions.com/ubuntu/erlang_solutions.asc | sudo apt-key add -

echo "deb https://packages.erlang-solutions.com/ubuntu focal contrib" | sudo tee /etc/apt/sources.list.d/rabbitmq.list

sudo apt update
sudo apt install erlang

curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.deb.sh | sudo bash
sudo apt install rabbitmq-server
$ systemctl status  rabbitmq-server.service
$ systemctl is-enabled rabbitmq-server.service 
sudo systemctl enable rabbitmq-server

sudo rabbitmqctl add_user admin StrongPassword
sudo rabbitmqctl set_user_tags admin administrator

rabbitmqctl add_vhost /my_vhost
rabbitmqctl set_permissions -p /myvhost user ".*" ".*" ".*"

```
