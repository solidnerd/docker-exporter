docker-engine-exporter
=====================

This repo is inspired from a slack post from [bvis][https://github.com/bvis] in the docker community slack how to export dynamically your engine metrics.


## Why does it work ?

The clue is that you export `docker_gwbridge`interface in the swarm cluster. and there is the engine accessible


## Setup a machine
For example do this for 3 nodes

```bash
docker-machine create --driver xhyve \
--xhyve-boot2docker-url=https://github.com/boot2docker/boot2docker/releases/download/v1.13.0-rc4/boot2docker.iso \
node01
```

## Determine docker_gwbridge ip address

```bash
ip -o -4 addr show docker_gwbridge | awk -F '[ /]+' '/global/ {print $4}'
````

In my case it was `172.18.0.1` . 

## Enable Metrics on your engine

```bash
cat > /etc/docker/daemon.json <<EOF
{
  "experimental":true,
  "metrics-addr":"0.0.0.0:5050"
}
EOF
```
After you did this restart or reload your engine .

## Setup your Terminal

Terminal 1: 
```bash
eval $(dm env node01)
```
Terminal 2: 
```bash
eval $(dm env node02)
```
Terminal 3: 
```bash
eval $(dm env node03)
```

## Lets build our services

```bash
docker build -t solidnerd/prometheus-docker-exporter prometheus
```

```bash
docker build -t solidnerd/socat socat
```

## Push them to a registry
This step depends on you


## Create you network and monitoring infrastructure

Create your monitoring network.

```bash
docker network create -d overylay monitoring
```

So expose your engine metrics in the swarm cluster.

```bash
docker \
  service create \
  --mode global \
  --name docker-exporter \
  --network monitoring \
  --env IN="172.18.0.1:5050" \
  --env OUT="5050" \
  --publish 5051:5050 \
  solidnerd/socat
```

Start your prometheus engine.

```bash
docker service create \
  --name prometheus \
  --network monitoring \
  --publish 9090:9090 \
  --constraint 'node.hostname==node01' \
solidnerd/prometheus-docker-exporter
```

