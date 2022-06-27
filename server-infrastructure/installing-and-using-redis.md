# Installing and Using Redis

## Installing Redis on Ubuntu 20.04 LTS

```shell
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
kill 3832


```

## Using Redis with Python

### Install redis-py

```
pip install redis
```

### Using Redis Lists (Preferred Method)

```python
import redis
import json

"""Create Redis Connection Pool"""
# Using TCP/IP Connection (use this method if unsure)
redis_pool = redis.ConnectionPool(host=config.redis_host, port=config.redis_port, db=0, password=config.redis_pw)

# Using Unix Domain Sockets
# redis_pool = redis.ConnectionPool(connection_class=redis.UnixDomainSocketConnection, path="/var/run/redis/redis-server.sock", password=config.redis_pw, db=0)

r = redis.Redis(connection_pool=redis_pool, charset="utf-8", decode_responses=True)

"""Push Message to Redis List"""
def redis_message(messages):
    for message in messages:
        r.rpush('list-name', json.dumps(message))
    return None

"""Receive Message from Websocket"""
def websocket_message_handler(ws, msg):
    messages = json.loads(msg)   
    redis_message(messages)
    return None
    
"""Start websocket client"""   
def main():
    my_client.start_stream_thread()
    return None
    
if __name__ == "__main__":
    main()
```

### Using Pub/Sub (Alternative Method)

```python
import redis 
import json

# Connect
r = redis.Redis(host=config.redis_host, port=config.redis_port, db=0,
                password=config.redis_pw, charset="utf-8", decode_responses=True)
p = r.pubsub()  # Activate Pubsub

# Subscribe
p.subscribe('my-first-channel')

#Publish Messages
r.publish('my-first-channel', 'some data')
# If message is JSON or Dicts then convert to string first
r.publish('my-first-channel', json.dumps(some_data))

while True
    try:
        message = p.get_message()
        if message:
            print('message received', message['data'])
            # If message was JSON, convert back
            msg = json.loads(message['data'])
            time.sleep(0.001)  # be nice to the system
            
# Unsubscribe
p.unsubscribe()
```

&#x20;
