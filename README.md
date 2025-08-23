
## Getting EC2 Ready!

This guide explains how to prepare an Ubuntu Server (EC2) for running containerized applications using Docker and docker-compose.
The goal: keep the machine minimal — only `Docker` and `docker-compose` installed.

### Docker

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu jammy stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update -y
    sudo apt install -y docker-ce docker-ce-cli containerd.io
    sudo groupadd docker
    sudo usermod -aG docker $USER
    newgrp docker
    sudo chown root:docker /var/run/docker.sock
    sudo chown -R root:docker /var/run/docker
    docker run hello-world

### Docker Compose

    VERSION=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | jq -r .tag_name)
    sudo curl -L "https://github.com/docker/compose/releases/download/$VERSION/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    docker-compose --version


### nginx-proxy

We use `jwilder/nginx-proxy` to automatically reverse-proxy apps by domain name.

    cd ~/apps
    git clone git@github.com:arxoft/nginx-proxied-dockerized-apps.git .
    docker network create proxy-net
    docker-compose -f docker-compose.yml up -d

Then start each child app independently: 

    ./restart <dir-name>

## Guidelines for Apps:

Each app is a separate dockerized repository cloned inside ~/apps/.

### Requirements for each app

- Must include these Compose files:
    - `docker-compose.yml`         	(base, all common stuff)
	- `docker-compose.prod.yml` 	(production override, with nginx-proxy stuff)
	- `docker-compose.dev.yml` 	    (development override, without nginx-proxy stuff)

The service that should be proxied:

- Must have its `container_name` match the repo name
- Must be attached to `proxy-net` network
- Must define `VIRTUAL_HOST` and `VIRTUAL_PORT` environment variables

Here's an example `docker-compose.prod.yml` of an app:

    version: "3.9"
    services:
    app:
        restart: always
        networks:
        - proxy-net
        - app-net
        environment:
        - VIRTUAL_HOST=bmsapi.maktg.com
        - VIRTUAL_PORT=7200
    mongo:
        restart: always
    networks:
    proxy-net:
        external: true


With this setup, adding a new app is as simple as cloning the repo into `~/apps/`, making sure its configs follow the conventions, and running `./restart <app-name>`.