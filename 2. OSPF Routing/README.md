## Commands for OSPF Routing
#### 1. Setting up Virtual Machines
Each of the Machines hostname needs to be changed: 
```console
hostname RX
```
And check if the hostname changed correctly by using:
```console
hostnamectl
--
Static hostname: ubuntu
Transient hostname: R1
```

#### 2. IP Address Assignment
To edit the netplan file use:
```console
sudo nano /etc/netplan/50-cloud-init.yaml
```
Edit the file as followed: **NO TABS!**
R1
```console
network:
    version: 2
    ethernets:
        ens2:
            dhcp4: no
            addresses: [172.16.0.1/24]
        ens3:
            dhcp4: no
            addresses: [10.0.1.1/30]
```
R2
```console
network:
    version: 2
    ethernets:
        ens2:
            addresses: [10.0.1.2/30]
            dhcp4: no
        ens3:
            addresses: [10.0.3.1/30]
            dhcp4: no
        ens4:
            addresses: [10.0.2.1/30]
            dhcp4: no
```
R3
```console
network:
    version: 2
    ethernets:
        ens2:
            addresses: [10.0.2.2/30]
            dhcp4: no
        ens3:
            addresses: [10.0.4.2/30]
            dhcp4: no
        ens4:
            addresses: [10.0.5.1/30]
            dhcp4: no
```
R4
```console
network:
    version: 2
    ethernets:
        ens2:
            dhcp4: no
            addresses: [10.0.3.2/30]
        ens3:
            dhcp4: no
            addresses: [10.0.4.1/30]
        ens4:
            dhcp4: no
            addresses: [10.0.255.1/30]
```
R5
```console
network:
    version: 2
    ethernets:
        ens2:
            dhcp4: no
            addresses: [10.0.5.2/30]
        ens3:
            dhcp4: no
            addresses: [192.168.1.1/24]
```
Client
```console
network:
    version: 2
    ethernets:
        ens2:
            dhcp4: no
            addresses: [172.16.0.100/24]
```
MITM
```console
network:
    version: 2
    ethernets:
        ens2:
            addresses: [10.0.255.2/30]
            dhcp4: no
```
After configuring the Routers and Users apply the netplan:
```console
netplan generate
netplan apply
```
use `ip addr` to check if the netplan applied correctly to each of the interfaces.
#### 3. BIRD for OSPF
First open the bird.conf file to edit it:
```console
sudo nano /etc/bird/bird.conf
```

