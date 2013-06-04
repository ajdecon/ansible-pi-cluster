Raspberry Pi Beowulf Cluster
----------------------------

Prep steps (by hand):
=====================

Head node:
- Initialize Raspbian
    - No desktop
    - OpenSSH on
    - Change password
- Copy image for other SD cards
- Set up SSH key
- Configure eth0:1 interface - statis 192.168.42.1
    - eth0 remains DHCP

Compute nodes:
- Set up SSH key
- Configure interfaces
