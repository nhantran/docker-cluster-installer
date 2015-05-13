# Docker Cluster Installation Guide
In order to install a Docker cluster, you may need to install a "backend" such as Consul first.
Assume that we have some following hosts:

host | IP | consul mode | docker mode
-----|----|-------------|------------
1 | 192.168.1.2   | server | node
2 | 192.168.1.3   | server | node
3 | 192.168.1.4   | client | node
4 | 192.168.1.5   | client | node
5 | 192.168.1.100 | N/A    | manager

The column "docker mode" means that we want to create a Docker cluster across first 4 nodes and the last node used for managing the cluster.

The column "consul mode" means that we are using the first 2 nodes as Consul servers and others as Consul clients.

All of these are forming a Consul cluster that will be used as a back-end of our Docker cluster

## Install Consul cluster
The cluster is to be used as a back-end for Docker cluster

#### For each host [192.168.1.2 -> 192.168.1.5], check out and install Consul:
```bash
      cd ~/
      git clone https://github.com/nhantran/docker-cluster-installer.git
      cd docker-cluster-installler
      sudo ./installConsul --bootstrap-ip "192.168.1.2,192.168.1.3"
```
#### Verify the installation on any hosts [192.168.1.2 -> 192.168.1.5]
```bash
      consul members
```
## Install Docker cluster
This cluster is using the recent installed Consul cluster above as back-end

#### For each host [192.168.1.2 -> 192.168.1.5], run the following command
```bash
      sudo ./installDocker --node --consul-ip 192.168.1.2
```
#### On the host [192.168.1.100], run the following command
```bash
      sudo ./installDocker --manager --consul-ip 192.168.1.2
```
#### Verify the installation on the "manager" host [192.168.1.100]
* List cluster info
```bash
      docker -H :2385 info
```
* List out containers on cluster:
```bash
     docker -H :2385 ps
```
* View log from each Cassandra container/service
```bash
      docker -H :2385 logs <container-id>
```
* Submit a "start nginx container" request
```bash
      docker -H :2385 run -d --name www -p 80:80 nginx:1.17.11
```
* Verify registered services
```bash
      curl http://192.168.1.2:8500/v1/catalog/services | python -m json.tool
      curl http://192.168.1.2:8500/v1/catalog/service/nginx-80 | python -m json.tool
```
