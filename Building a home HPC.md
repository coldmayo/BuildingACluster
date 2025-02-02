The goal is to build my own Beowulf cluster
- Probably get a bunch of cheap refurbished PCs
- Why? Because it would be cool
Parts needed (hardware wise):
- Head node
	- The computer that relays instructions to the rest of the computers (compute nodes) across an isolated local network
	- Needs at least two Ethernet ports, one for the outside world, and another for the isolated internal compute node network
		- Could be achieved from the Networking Switch
- RAID (Redundant Array of Independent Disks)
	- storage system that uses multiple hard drives to store data
	- Use old PC + large capacity HDDs/SDDs to add storage space
		- Use OpenMediaVualt as NAS software
- Compute nodes
	- Computers that get instructions from the head node, carry out the tasks, and then sends the results back to the head node
- Networking Switch(es)
	- connects devices in a network to each other, enabling them to talk by exchanging data packets
	- Network switch needs to have at least as many ports as you have compute nodes, but more would be nice
- Networking Cables
- Rack and mounting hardware
	- Looking at the 12U rack at the moment
- Power strips
	- For smaller clusters consumer grade power strips should be adequate
- Uninterruptible Power Supply
	- provides backup power, protecting equipment from damage in the event of grid power failure
- KVM switch and cables
	- A KVM switch is used for controlling, switching between, and manage multiple PCs or servers via a single keyboard, monitor and mouse
- Rack Mounts
	- For our case, we probably only need 2U racks for now

Software needed:
- OS
	- Probably Debian or CentOS
- MPI (Message Passing Interface)
	- Facilitates communication between nodes
	- Should use OpenMPI (could roll out my own someday?)
- Resource Management:
	- Deals with scheduling
	- Probably use SLURM (could roll out my own someday?)
- Set up SSH
- OpenMediaVualt
	- NAS Software
- Cluster Monitoring Suite
	- Ganglia
- Package manager
	- Yum
- File System for the Network (NFS)
- Firewall
	- Firewalld
- NTP
	- configure Network Time Protocol (NTP) on a cluster of servers

The basic steps:
1. Install OS on each machine
	1. Assign static IP addresses to all nodes for easy identification
2. Configure head node
	1. Set up shared directories (eg /home/cluster) that is accessible by all nodes
	2. Generate SSH keys on the head node and copy the public key to each compute node
	3. Use /etc/hosts to map hostnames to IP addressess for each node
3. Install Cluster software
	1. Install MPI libraries
	2. Configure job scheduling software
4. Configure Compute Nodes
	1. Mount NFS shared directories