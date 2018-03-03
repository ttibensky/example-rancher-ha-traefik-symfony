# Example rancher 2.0 playground

Example rancher 2.0 in high availability with traefik proxy and sample symfony apps deployed.

## Setup guide

### Requirements

- docker
- docker-compose
- docker-machine
- virtualbox
- rancher-compose http://rancher.com/docs/rancher/v1.0/en/rancher-compose/

### Create & start rancher server

```bash
docker-compose up -d
```

### Create virtual machines & register them as hosts inside rancher server

- create virtual machine running rancher os `docker-machine create -d virtualbox --virtualbox-boot2docker-url https://releases.rancher.com/os/latest/rancheros.iso rancher-node-1`
- go to http://127.0.0.1:8080/env/1a5/infra/hosts
- run `ifconfig` and look for `vboxnet0`, copy its IP address, you will need it in next step. mine was `192.168.99.1`
- use `http://192.168.99.1:8080` as `Host registration url
- ssh into docker-machine `docker-machine ssh rancher-node-1`
- run the rancher agent docker command inside your docker-machine, it looks like this `sudo docker run --rm --privileged -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/rancher:/var/lib/rancher rancher/agent:v1.2.9 http://192.168.99.1:8080/v1/scripts/C121D25127D8960B39A7:1514678400000:OcQj1Mj92vzFajR6JSoQ0qg`
- in a moment you should see your host pop up here http://127.0.0.1:8080/env/1a5/infra/hosts
- repeat these steps with `rancher-node-2` and `rancher-node-3`, replace `1` for respective numbers

A shorthand commands:

```bash
docker-machine create -d virtualbox --virtualbox-boot2docker-url https://releases.rancher.com/os/latest/rancheros.iso rancher-node-1
docker-machine ssh rancher-node-1
sudo docker run --rm --privileged -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/rancher:/var/lib/rancher rancher/agent:v1.2.9 http://192.168.99.1:8080/v1/scripts/C121D25127D8960B39A7:1514678400000:OcQj1Mj92vzFajR6JSoQ0qg
exit

docker-machine create -d virtualbox --virtualbox-boot2docker-url https://releases.rancher.com/os/latest/rancheros.iso rancher-node-2
docker-machine ssh rancher-node-2
sudo docker run --rm --privileged -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/rancher:/var/lib/rancher rancher/agent:v1.2.9 http://192.168.99.1:8080/v1/scripts/C121D25127D8960B39A7:1514678400000:OcQj1Mj92vzFajR6JSoQ0qg
exit

docker-machine create -d virtualbox --virtualbox-boot2docker-url https://releases.rancher.com/os/latest/rancheros.iso rancher-node-2
docker-machine ssh rancher-node-2
sudo docker run --rm --privileged -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/rancher:/var/lib/rancher rancher/agent:v1.2.9 http://192.168.99.1:8080/v1/scripts/C121D25127D8960B39A7:1514678400000:OcQj1Mj92vzFajR6JSoQ0qg
exit
```

### Build and push symfony example app to dockerhub.com

This step is here only for a reference, it was already done.

```bash
docker login
docker build . -t ttibensky/rancher-symfony-example-1
docker push ttibensky/rancher-symfony-example-1
```

### Deploy sample symfony app

- go to http://127.0.0.1:8080/env/1a5/api/keys, create a key and replace key and secret in the command below
- you can also have the key and secret set as environment variables, see http://rancher.com/docs/rancher/v1.0/en/rancher-compose/

```bash
cd symfony-example-1
rancher-compose --url http://127.0.0.1:8080/v1 --access-key 7C54CB113CBDAA4DFA1D --secret-key b3PVS8emjJj3CFqDaW3agY7mwV4wx6WJRLWRoKdR \
    -f docker-compose.yml -f rancher-compose.yml -p symfony-example-1 up -d --force-upgrade --confirm-upgrade
```

### Sources

- http://rancher.com/docs/rancher/latest/en/hosts/custom/
- http://rancher.com/docs/rancher/v1.0/en/installing-rancher/installing-server/multi-nodes/
- http://rancher.com/docs/rancher/v1.0/en/rancher-compose/
- https://gist.github.com/lmmendes/fbed32a452cf02d2a1095658795cb3d2

### Known issues

- all hosts are having some issues with ipsec, healthcheck and scheduler stacks https://github.com/rancher/rancher/issues/7664
- rancher-compose deploy dowsn't work `ERRO[0000] Failed to open project symfony-example-1: Can not create a stack, check API key [7C54CB113CBDAA4DFA1D] for [http://127.0.0.1:8080/v1]
                                       FATA[0000] Failed to read project: Can not create a stack, check API key [7C54CB113CBDAA4DFA1D] for [http://127.0.0.1:8080/v1]`
