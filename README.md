# ansible-docker-swarm 

Setup Docker Swarm with Ansible.

In this setup I have a client node, which will be my jump box, as it will be used to ssh with the docker user to my swarm nodes with passwordless ssh access.

## Pre-Check

Hosts file: 

```
$ cat /etc/hosts
10.0.156.253 client #for jump host or ansible server (need the same OS with swarm)
10.0.159.228 swarm-worker-1
10.0.139.153 swarm-worker-2
10.0.129.62 swarm-manager
10.0.129.62 swarm-manager-1
10.0.142.174 swarm-manager-2
10.0.135.236 swarm-manager-3
```

Install Ansible:

```
$ apt install python-setuptools -y
$ easy_install pip
$ pip install ansible
```

Ensure passwordless ssh is working:

```
$ ansible -i inventory.ini -u ubuntu --private-key=super8.pem -m ping all
client | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
swarm-manager | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
swarm-manager-2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
swarm-manager-3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
swarm-worker-2 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
swarm-worker-1 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

## Deploy Docker Swarm

```
$ ansible-playbook -i inventory.ini -u ubuntu --private-key=no8.pem deploy-swarm.yml
PLAY RECAP 

client                     : ok=11   changed=3    unreachable=0    failed=0   
swarm-manager              : ok=18   changed=4    unreachable=0    failed=0   
swarm-manager-2            : ok=18   changed=4    unreachable=0    failed=0
swarm-manager-3            : ok=18   changed=4    unreachable=0    failed=0
swarm-worker-1             : ok=15   changed=1    unreachable=0    failed=0   
swarm-worker-2             : ok=15   changed=1    unreachable=0    failed=0   
```

SSH to the Swarm Manager and List the Nodes:

```
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
73qtm32qqvluterazqnj3zpfz *   ip-10-0-129-62      Ready               Active              Leader              19.03.3
m3d2cncwidhplzcf1azqlgpu1     ip-10-0-135-236     Ready               Active              Reachable           19.03.3
s9cusb1y4pco8qnidyw1gn88z     ip-10-0-139-153     Ready               Active                                  19.03.3
u6q3mkpwq9trrclaf9o77gtdp     ip-10-0-142-174     Ready               Active              Reachable           19.03.3
cijpvapsvvolm03r42e5zahom     ip-10-0-159-228     Ready               Active                                  19.03.3
```

Example for creating a Nginx Demo Service:

```
$ docker network create --driver overlay appnet
$ docker service create --name nginx --publish 80:80 --network appnet --replicas 6 nginx
$ sudo docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
i7uknqumr7bw        nginx               replicated          6/6                 nginx:latest        *:80->80/tcp

$ sudo docker service ps nginx
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
dn3b90vuhuk1        nginx.1             nginx:latest        ip-172-31-35-128    Running             Running 35 seconds ago                       
weeu3kxhdyc5        nginx.2             nginx:latest        ip-172-31-39-81     Running             Running 35 seconds ago                       
e3krkes9kx4e        nginx.3             nginx:latest        ip-172-31-10-112    Running             Running 33 seconds ago                       
wpfvb94ai32u        nginx.4             nginx:latest        ip-172-31-35-128    Running             Running 34 seconds ago                       
hyaszjsnygyj        nginx.5             nginx:latest        ip-172-31-39-81     Running             Running 35 seconds ago                       
0pi0afczqxvu        nginx.6             nginx:latest        ip-172-31-10-112    Running             Running 32 seconds ago                       
```

## Delete the Swarm:

```
$ ansible-playbook -i inventory.ini -u ubuntu --private-key=no8.pem delete-swarm.yml

PLAY RECAP 
swarm-manager              : ok=2    changed=1    unreachable=0    failed=0   
swarm-manager-2            : ok=2    changed=1    unreachable=0    failed=0
swarm-manager-3            : ok=2    changed=1    unreachable=0    failed=0
swarm-worker-1             : ok=2    changed=1    unreachable=0    failed=0   
swarm-worker-2             : ok=2    changed=1    unreachable=0    failed=0   
```

Ensure the Nodes is removed from the Swarm, SSH to your Swarm Manager:

```
$ docker node ls
Error response from daemon: This node is not a swarm manager. Use "docker swarm init" or "docker swarm join" to connect this node to swarm and try again.
```
