---
description: >-
  This guide will walk through the process of developing a FASTAPI and deploying
  to an AWS EC2 instance with a custom domain and SSL/HTTPS certification.
---

# Creating an API with FastAPI

### Create a FASTAPI project

```
pip install fastAPI
```

### Create main.py

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

### Launch API

```shell
# Launch Uvicorn and load main.py app
sudo /home/ubuntu/fastapi/env/bin/uvicorn 
sudo /home/ubuntu/fastapi/main:app --reload
```

### Visit Documentation

### Creating a FastAPI Ubuntu service

```bash
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
 sudo systemctl start fastapi.servicebas
```

### \[Optional] Create a Monitor with Monit

<pre class="language-bash"><code class="lang-bash"># Create a new Monit file:
sudo nano /etc/monit/conf.d/fastapi.conf

<strong># Add the following to the new file:
</strong><strong>check process fastapi.service
</strong>    matching "fastapi.service"
    start program = "/bin/systemctl start fastapi.service"
    stop program = "/bin/systemctl stop fastapi.service"
    if not exists then alert
    set log /var/log/fastapi.log

# Reload
sudo monit reload

# Restart Monit
sudo service monit restart</code></pre>

### Websockets

```python
from fastapi import FastAPI, WebSocket
import random

# Create application
app = FastAPI(title='WebSocket Example')

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    print('Accepting client connection...')
    await websocket.accept()
    while True:
        try:
            # Wait for any message from the client
            await websocket.receive_text()
            # Send message to the client
            resp = {'value': random.uniform(0, 1)}
            await websocket.send_json(resp)
        except Exception as e:
            print('error:', e)
            break
    print('Bye..')
```
