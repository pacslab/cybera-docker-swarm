# Docker Swarm on Cybera Cloud

This repository includes tutorials about running and testing Docker Swarm mode
on the Cybera cloud. In this tutorial, we assume that you already have a Cybera
account and you have the quota to create at least 2 VMs which could be clustered together.

- To get access to the Cybera cloud request for an account [here](https://rac-portal.cybera.ca).
  - Choose the option to sign in using your institution account.
- You can login to the portal after getting access [here](https://cloud.cybera.ca/auth/login/).
- You need network access to you VMs (they have internal IP addresses).
  - To do so, install [OpenVPN](https://openvpn.net/) version 2.4.6.
  - Follow the instructions from [here](https://wiki.cybera.ca/display/RAC/Rapid+Access+Cloud+Virtual+Private+Network) to connect
    to the internal network.


# Create Virtual Machines

- Create 3 VMs with "m1.small" flavor, instance boot source set to boot from image
and image name of "Ubuntu 18.04".
- In the access and security section, create a new key pair if you don't already have one.
- Wait for the VMs to boot up and be available.
- Ping your VMs to make sure you have network access to them (you might need to enable
  ICMP access in the network security groups section). Also allow TCP port 22 for ssh
  connection.

```
ping 10.1.X.X

ssh -i PRIVATE_KEY_FILE ubuntu@10.1.X.X
```

# Install Docker
```
sudo apt-get update && sudo apt-get -y install docker.io
sudo apt-get install -y apt-transport-https
sudo swapoff -a
sudo systemctl enable docker
```

Then finalize the installation (to enable non-sudo user to run docker commands):

```
docker ps
sudo docker ps

sudo usermod -aG docker $USER
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "/home/$USER/.docker" -R
sudo chown "$USER":"$USER" /var/run/docker.sock
sudo chmod g+rwx /var/run/docker.sock -R
sudo systemctl enable docker
```

# Create a Cluster for Swarm
For creating this section, we use [this tutorial](https://rominirani.com/docker-swarm-tutorial-b67470cf8872). You can
follow that one or just go through the commands that we have here.

First, select one of the VMs to be the swarm manager. Run this command on the manager:
```
docker swarm init --advertise-addr MANAGER_IP
```

You would see an output like this:
```
ubuntu@demo-1:~$ docker swarm init --advertise-addr 10.1.13.200
Swarm initialized: current node (pim3mrnkpuil1bmz22zk47ta5) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1p2uxmaslgwpqpqoh235vrsgs760krohb8h4m8lbgucbooyxsr-ahga65ag49n796e2xs0uqeru8 10.1.13.200:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

This basically tells us two things. First of all, the cluster has been initialized with success. Second, it gives us a command to
run on the other nodes to join the cluster. Make sure to open the port 2377 on your network security group so that workers can
contact manager on that port. Running this command on the workers would produce an output like this:

```
ubuntu@demo-2:~$ docker swarm join --token SWMTKN-1-1p2uxmaslgwpqpqoh235vrsgs760krohb8h4m8lbgucbooyxsr-ahga65ag49n796e2xs0uqeru8 10.1.13.200:2377
This node joined a swarm as a worker.
```

Now you can see a list of nodes that have joined the cluster:

```
ubuntu@demo-1:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
pim3mrnkpuil1bmz22zk47ta5 *   demo-1              Ready               Active              Leader              18.06.1-ce
u0ec1dq0xapvr7065ehk2737v     demo-2              Ready               Active                                  18.06.1-ce
plgix2rzgicdnhcfd92vapgyt     demo-3              Ready               Active                                  18.06.1-ce
```

This means you a working cluster that you can use to deploy services and other stuff on.

# Docker Services
## Create a Service

To create a service, run the following command:

```
docker service create --replicas 5 -p 80:80 --name web nginx
```

After a few seconds, 5 instances of the nginx image should be up and running. Get a list of services on this cluster:

```
docker service ls
```

The output should be similar to this one:

```
ubuntu@demo-1:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
togwgnzoyv9g        web                 replicated          5/5                 nginx:latest        *:80->80/tcp
```

Open up the TCP port 80 on your network security
group to be able to access the resulting webpage.
Now, open up a browser and go to the following address:

```
http://VM1_IP:80
```

Now, try out other VMs IPs. As you can see, it doesn't matter which IP you use,
you will see the same results. This is because docker creates a virtual network
in the cluster and load balances the traffic between different VMs that have
the same image running on them. Let's take a look at the resulting containers:

```
ubuntu@demo-1:~$ docker service ps web
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
2zrudha3t6l2        web.1               nginx:latest        demo-2              Running             Running 6 minutes ago
rg8aqd16q1mm        web.2               nginx:latest        demo-3              Running             Running 6 minutes ago
j02oztr36mze        web.3               nginx:latest        demo-2              Running             Running 6 minutes ago
di44u33sciel        web.4               nginx:latest        demo-3              Running             Running 6 minutes ago
o7vcojo611lz        web.5               nginx:latest        demo-1              Running             Running 6 minutes ago
```

As you can see docker automatically distributes your running instances among different VMs to make sure
you have a good availability even if some of your nodes go down.

## Scaling up/down

```
docker service scale web=8
```

After a few seconds, you instances should increase to 8:

```
ubuntu@demo-1:~$ docker service ps web
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
2zrudha3t6l2        web.1               nginx:latest        demo-2              Running             Running 10 minutes ago
rg8aqd16q1mm        web.2               nginx:latest        demo-3              Running             Running 10 minutes ago
j02oztr36mze        web.3               nginx:latest        demo-2              Running             Running 10 minutes ago
di44u33sciel        web.4               nginx:latest        demo-3              Running             Running 10 minutes ago
o7vcojo611lz        web.5               nginx:latest        demo-1              Running             Running 10 minutes ago
904dzwvxr9ww        web.6               nginx:latest        demo-1              Running             Running 26 seconds ago
gg2r9s2mq2ay        web.7               nginx:latest        demo-3              Running             Running 30 seconds ago
lxx8fnhknjps        web.8               nginx:latest        demo-1              Running             Running 27 seconds ago
```

You could also inspect the nodes:

```
docker node inspect self
docker node inspect demo-2
```

## Deleting the service

```
docker service rm web
```

# Create a Stack

# Clear Everything
