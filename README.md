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

Install virtualenv
```
apt install virtualenv -y
```

Install pip
```
apt install pip -y
```

Install kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl
sudo mv kubectl /usr/local/bin
```


Install GitLab using Docker Compose
```
mkdir gitlab
cd gitlab

export GITLAB_HOME=$(pwd)/gitlab

vi docker-compose.yml
---
# docker-compose.yml
version: '3.7'
services:
  web:
    image: 'gitlab/gitlab-ce:latest'
    restart: always
    hostname: 'localhost'
    container_name: gitlab-ce
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://localhost'
    ports:
      - '8080:80'
      - '8443:443'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    networks:
      - gitlab
  gitlab-runner:
    image: gitlab/gitlab-runner:alpine
    container_name: gitlab-runner    
    restart: always
    depends_on:
      - web
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - '$GITLAB_HOME/gitlab-runner:/etc/gitlab-runner'
    networks:
      - gitlab

networks:
  gitlab:
    name: gitlab-network
---

# In the browser
<IP-DEVOPS-HOST>:8080

# Get the password
docker exec -it gitlab-ce grep 'Password:'
```




## Install the Kubernetes Cluster with Kubespray

```
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray

virtualenv venv
source venv/bin/activate

pip3 install -r requirements.txt

cp -rfp inventory/sample inventory/mycluster

declare -a IPS=(172.16.0.153 172.16.0.167 172.16.0.213)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --user=root cluster.yml
```

```
mkdir ~/.kube
scp root@<NODE1-IP>:/etc/kubernetes/admin.conf .
mv admin.conf config

# edit IP in config from 127.0.0.1 to IP from Node1
```


## Install the Monitoring Host

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


Install Grafana and Loki with docker-compose
```
wget https://raw.githubusercontent.com/grafana/loki/main/production/docker-compose.yaml
docker-compose up -d

# In the browser
<IP-MONITORINGHOST>:3000

# User:Pass admin:admin
```



