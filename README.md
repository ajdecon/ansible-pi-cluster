Ansible scripts for my Raspberry Pi Cluster
===========================================

This is a small collection of 
[Ansible](http://ansible.cc) playbooks for my Raspberry Pi cluster.

**These really aren't set up to be general at the moment** -- i.e., there
are still hard-coded IPs and names in config files, and other crap -- but
if you want to try to use this to build your own cluster, feel free to give it
a shot. :)  It should at least be pretty straightforward to read and understand
the playbooks.

Ansible is a really cool tool for automating system configuration tasks, so 
I automated most of the software configuration with Ansible playbooks.
However, I did all the OS provisioning and networking manually.
This set of playbooks will set up an HPC cluster with some of the following 
libraries and services:

* Cluster Scheduler: [SLURM](http://slurm.schedmd.com)
* Message-passing library: [OpenMPI](http://www.open-mpi.org/)
* Benchmark: HPCC
* Shared filesystem: NFS
* Time serever: NTP
* DNS: dnsmasq
* Performance montioring: Ganglia

**Important**: the notes below describe roughly what I did, but there was a lot of 
trial-and-error and I didn't note all the blind alleys.  I haven't reproduced
with this set of instructions yet, so you might have to troubleshoot a bit.

Physical setup and OS
---------------------

1. Get 2 or more Raspberry Pi Model B's. One will be the head node, the others 
will be compute nodes. 
1. Download the most recent [Raspbian Linux](http://www.raspbian.org/) and copy
the raw image onto an SD card. (I used the "dd" command on a Linux laptop to do
this, but [this page](http://www.raspbian.org/RaspbianInstaller) has additional 
information on how to do that.
1. I booted up the SD card on one of the Raspberry Pi's. I changed the password,
made sure SSH was turned on and X Windows was turned off, then shut it back down.
1. Copy the contents of the Raspbian SD card to N idential SD cards for the 
other Raspberry Pi's.


Networking setup
----------------

I set up all my Raspberry Pi's with static addresses in the 192.168.42.0/24 subnet,
and gave the head node a second interface to DHCP.

Mostly I did this so that I could run the cluster "stand-alone", i.e. without an
external DHCP server, but so that I could also plug into a larger network
and have the head node reachable from the rest of the network.

Also because HPC clusters usually keep all their compute nodes on a "private" network,
and I was trying to make it look like a "real" cluster. :)

I probably could have figured out how to do this with Ansible, but I did this 
by hand at the time.

1. Get an ethernet switch and connect all the Raspberry Pi's. Also temporarily
connect this switch to your home router, or other DHCP server.
1. Turn on all the Raspberry Pi's and let them boot.
1. Use nmap or look at your router config to figure out what their temporary IPs are.
1. SSH into the head node and set up two interfaces on eth0.  I configured eth0 to have
the static address 192.168.42.1 and eth0:1 to DHCP, so I could have the head node NAT 
to the outside world when connected to my home router. I also changed the head node
hostname to "pihead" at this point.
1. SSH into each "compute" Raspberry Pi and configure a static IP on eth0. Change the 
hostname to something like "pi01", "pi02", etc. Then reboot.
1. Create an /etc/hosts file on the head node to list the compute node names and IPs.

At this point it's also a good idea to set up passwordless SSH with keys from your laptop 
into the head node, and from the head node to all the compute nodes. I used the "pi" 
user for this and gave "pi" sudo rights on all nodes.


Ansible configuration of head node
----------------------------------

If you haven't installed [Ansible](http://ansible.cc) yet, you should do
so! Run "pip install ansible" or "easy_install ansible" to do this.

Edit the "hosts.pi" file and enter the IP address of the head node, and the 
names or IPs of the compute nodes. 

You should (hopefully) be able to set up all the cluster services using the following
command:

    ansible-playbook -i hosts.pi headnode-main.yml

Ansible configuration of compute nodes
--------------------------------------

The compute nodes are on a private network, so you have to configure them from the 
head node.

SSH into the head node as "pi" and confirm that this repo is present as "ansible-pi-cluster".
Then you should be able to just run

    ansible-playbook -i hosts.pi computes-main.yml

And with luck, you should then have your SLURM cluster!

For information about how to use SLURM, see the 
[quickstart guide](http://slurm.schedmd.com/quickstart.html).
