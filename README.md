# The Hard Way Night

## Nothing todo and no new Netflix series to view?

Lets have some fun and create a quick full cluster setup

We will deploy and configure following:

DevOps Node:
- Gitlab CI in docker
  
Monitoring Node:
- Grafana
- Loki

Kubernetes 3 Node Cluster:
- Ingres-Controller
- Cert-Manager
- MetalLB


## DevOps Node

Install Ansible
```
sudo apt-add-repository ppa:ansible/ansible

sudo apt update
sudo apt install ansible

```

Install docker
```
curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh
```

Install docker-compose
```
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.18.1/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose

chmod +x ~/.docker/cli-plugins/docker-compose
cp ~/.docker/cli-plugins/docker-compose /usr/bin/
docker-compose --version
```

