# Intro

This guide will teach you how I personally setup a computing clusters with the following materials:
- 3 PCs (1 head node, 2 compute nodes)
- 4 Ethernet cables
- 1 network switch
This will also be fully LAN since my ISP does not allow IP forwarding (count your days Elauwit)

All of the computers should be connected to the network switch via ethernet cable and the network switch should be connected to a router or something.

# Installing Software

## CentOS

I chose to use CentOS Stream 10 for this build, Ubuntu is also common from what I've seen. If you want a guide for that specifically, there's plenty online already. I would suggest choosing an OS that is stable, you won't want to spend too much time on maintinence.

I think a lot of Linux distros come with a disk image writer preinstalled but otherwise you could use Etcher to put the ISO file on the USB.

The only things needing to be changed are:
- Automatic Partitioning, make sure to free up as much space as you can
- Set the name to something like head, node1, node2 (whatever it corresponds to)
- Set a root password, I set them all to the same one because it's easy to remember

## Packages needed

Since IP forwarding was not allowed for me, I had to download all of the required packages before setting up the static IPs. Here is a list of what you'll need on each node:
- nfs-utils
- openmpi
- openmpi-devel
- gcc
- gcc-c++
- make

I did this using yum, but dnf should work as well.

If you forget something or want to add a package after setting up the network, you could save the packages to tar.gz file on a usb and go from there.

# Networking

You can choose whatever IPs for this (as long as they fall in range) but I chose:
- Head Node: 192.168.1.100
- Node 1: 192.168.101
- Node 2: 192.168.102

Since CentOS uses Network Manager, I used nmcli for coniguring everything

## Head Node

```bash
nmcli con mod "Wired connection 1" ipv4.addresses 192.168.1.100/24
nmcli con mod "Wired connection 1" ipv4.gateway <put ip address of your router>
nmcli con mod "Wired connection 1" ipv4.dns 8.8.8.8
nmcli con mod "Wired connection 1" ipv4.method manual
nmcli con up "Wired connection 1"
```

Basically, set the static IP to be 192.168.1.100/24, the gateway will be the ip to your router especially if you want to actually access the internet. Method is set to manual because it prevents DHCP (Dynamic Host Configuration Protocol) from changing the IP and messing up communication.

We also have to allow IP forwarding so the head node can access the internet (In theory, Elauwit...) and then the compute nodes will set the head node's IP as their gateway to gain internet as well.

Edit ```/etc/sysctl.conf``` and write: ```net.ipv4.ip_forward = 1``` in the file and then run:
```bash
sudo sysctl -p
```
To apply the changes.

Personally, I shut the firewall off completely for the nodes, since it will only be running locally anyway for now:
```bash
sudo systemctl disable firewalld
```
Finally we need to update ```/etc/hosts``` file, which will allow linking names to ip addresses and put the following inside of it:
```
192.168.1.100 master
192.168.1.101 node1
192.168.1.102 node2
```

## Compute nodes

Every other node will need to have static IPs configured like the following:
```bash
nmcli con mod "Wired connection 1" ipv4.addresses 192.168.1.10X/24
nmcli con mod "Wired connection 1" ipv4.gateway 192.168.1.100
nmcli con mod "Wired connection 1" ipv4.dns 8.8.8.8
nmcli con mod "Wired connection 1" ipv4.method manual
nmcli con up "Wired connection 1"
```
Where the X is whatever number you want.

Just like the head node, disable the firewall and edit ```/etc/hosts```.

Make sure to ```ping``` the other nodes to make sure everything is being set up correctly.

# Set up SSH

First thing's first, set up a user (same user as the head node) on the other nodes:

```bash
sudo useradd -m <name>
sudo passwd <name>
```

In order for MPI to work correctly, we need to setup passwordless ssh between the nodes. So on the head node:
```bash
ssh-keygen -t rsa
ssh-copy-id node1
ssh-copy-id node2
```

Then test it out:
```bash
ssh node1
ssh node2
```

If things aren't working correctly check if ssh is working on each node:

```bash
sudo systemctl status sshd
```

# Set up NFS

Now we have to set up NFS which is a shared file system between all nodes + head node.

How I set this up was I had a folder called shared located at /home/head_node_user/shared at the head node while mounting /shared to /mnt. Thankfully you can sync the the /mnt/shared and /home/user_name/shared using fstab. You have to do this otherwise MPI gets confused, it isn't pretty but it seemed to work.

In the head node we need to edit the ```/etc/exports``` file by adding:
```
/home/user_name/shared 192.168.1.0/24(rw,sync,no_root_squash)
```

Once this is done, start up the server:

```bash
sudo systemctl enable nfs-server
sudo systemctl start nfs-server
```

Now we need to mount /shared onto the other nodes.

To do so do this on each node:

```bash
sudo mkdir /mnt/shared
sudo mount 192.168.1.100:/home/user_name/shared /mnt/shared
```

Add it to fstab to make it perminant:
```bash
sudo vim /etc/fstab
```
Add the following line:
```
192.168.1.1:/home/user_name/shared /mnt/shared nfs defaults 0 0
```

You can test this with: ```sudo mount -a```.

On the head node you can sync the /home/user/shared and /mnt/shared folder by editing its fstab:
```
/home/user/shared  /mnt/shared  none  bind  0  0
```

# Configure MPI

For some reason on CentOS OpenMPI is not installed in the path they expect. So you have to add these two lines to the ```.bashrc``` of your head node and to your compute nodes:

```
export PATH=$PATH:/usr/lib64/openmpi/bin

```

Then run ```source .bashrc``` to establish the changes. When you do it on each node make sure you are sshing from the head node and editting the bashrc from there, since that is how MPI will be accessing said nodes.

# Test MPI

Now you should be able to test out MPI. Go to the ```/shared``` directory on the head node and make a program like this:

```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    MPI_Init(NULL, NULL);
    int world_size;
    MPI_Comm_size(MPI_COMM_WORLD, &world_size);
    int world_rank;
    MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
    printf("Hello from process %d of %d\n", world_rank, world_size);
    MPI_Finalize();
    return 0;
}
```

Make a hostfile which lets MPI know where to look for the nodes. I named mine ```hosts.txt``` and this was what was in it:
```
master
node1
node2
```

But instead of the alias's you could put in the actual IPs if you wanted to:
```
192.168.1.100
192.168.1.101
192.168.1.102
```

You can compile and run this script like so:
```bash
mpicc hello_mpi.c -o hello_mpi
mpirun -np 4 --host hosts.txt ./hello_mpi
```

And now you should have a working computing cluster!
