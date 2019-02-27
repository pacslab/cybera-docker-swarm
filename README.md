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
  ICMP access in the network security groups section).

```
ping 10.1.X.X

ssh -i PRIVATE_KEY_FILE ubuntu@10.1.X.X
```

# Install Docker

# Create a Cluster for Swarm

# Create a Service

# Create a Stack

# Clear Everything
