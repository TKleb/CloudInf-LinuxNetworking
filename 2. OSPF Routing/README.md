## Commands for OSPF Routing
This cookbook is just to follow along with each step of the lab. For more information regarding them have a look into our report in the Documents folder.
#### 1. Setting up Virtual Machines
---
Each of the Machines hostname needs to be changed: 
```console
sudo hostnamectl set-hostname "hostname"
```
In addition to make the hostname persist after a reboot edit the file /etc/cloud/cloud.cfg and set the variable "preserve_hostname" to true. Relog into the Linux System to see the right hostname in your terminal: "ins@hostname"

And check if the hostname changed correctly by using:
```console
hostnamectl
-- output --
Static hostname: "hostname"

```

#### 2. IP Address Assignment
---
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
        lo:
            adresses: [192.168.2.1/24]
            dhcp4: no
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
        lo:
            adresses: [192.168.3.1/24]
            dhcp4: no
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
            routes:
                - to: 0.0.0.0/0
                  via: 172.16.0.1
```
MITM
```console
network:
    version: 2
    ethernets:
        ens2:
            addresses: [10.0.255.2/30]
            dhcp4: no
            routes:
            - to: 0.0.0.0/0
              via: 10.0.255.1
```
After configuring the Routers and Users apply the netplan:
```console
netplan generate
netplan apply
```
use `ip addr` to check if the netplan applied correctly to each of the interfaces.
#### 3. BIRD for OSPF
---
First open the bird.conf file to edit it:
```console
sudo nano /etc/bird/bird.conf
```
Edit the files as followed: You can either   
R1
```console
protocol device {
    scan time 10;
}

protocol kernel {
    metric 64;         
    import all;
    learn;
    export all; 
    persist;    
}

router id 1.1.1.1;

protocol ospf MyOSPF {
  tick 2;
  area 0 {
    stub no;
    interface "ens2" {
      stub;
      hello 15;
      retransmit 6;
      cost 10;
      transmit delay 5;
      dead count 3;
      wait 50;
      type broadcast;
    };
    interface "ens3" {
      hello 15;
      retransmit 6;
      cost 10;
      transmit delay 5;
      dead count 3;
      wait 50;
      type ptp;
    };
  };
}  
```
R2
```console
protocol device {
    scan time 10;
}

protocol kernel {
    metric 64;         
    import all;
    learn;
    export all;
    persist;     
}

router id 2.2.2.2;

protocol ospf MyOSPF {
  tick 2;
  area 0 {
    stub no;
    interface "ens2", "ens3", "ens4", "lo" {
      hello 15;
      retransmit 6;
      cost 10;
      transmit delay 5;
      dead count 3;
      wait 50;
      type broadcast;
  };
}
```
R3
```console
protocol device {
    scan time 10;
}

protocol kernel {
    metric 64;         
    import all;
    learn;
    export all;
    persist;     
}

router id 3.3.3.3;

protocol ospf MyOSPF {
  tick 2;
  area 0 {
    stub no;
    interface "ens2", "ens3", "ens4", "lo" {
      hello 15;
      retransmit 6;
      cost 10;
      transmit delay 5;
      dead count 3;
      wait 50;
      type broadcast;
  };
}
```
R4
```console
protocol device {
    scan time 10;
}

protocol kernel {
    metric 64;         
    import all;
    learn;
    export all;
    persist;     
}

router id 4.4.4.4;

protocol ospf MyOSPF {
  tick 2;
  area 0 {
    stub no;
    interface "ens2", "ens3" {
      hello 15;
      retransmit 6;
      cost 10;
      transmit delay 5;
      dead count 3;
      wait 50;
      type broadcast;
    };
    interface "ens4" {
      stub;
      hello 15;
      retransmit 6;
      cost 10;
      transmit delay 5;
      dead count 3;
      wait 50;
      type ptp;
    };
  };
}
```
R5
```console
protocol device {
    scan time 10;
}

protocol kernel {
    metric 64;         
    import all;
    learn;
    export all; 
    persist;    
}

router id 5.5.5.5;

protocol ospf MyOSPF {
  tick 2;
  area 0 {
    stub no;
    interface "ens2" {
      hello 15;
      retransmit 6;
      cost 10;
      transmit delay 5;
      dead count 3;
      wait 50;
      type broadcast;
    };
    interface "ens3" {
      stub;
      hello 15;
      retransmit 6;
      cost 10;
      transmit delay 5;
      dead count 3;
      wait 50;
      type ptp;
   };
  };
}
```
Don't forget to delete the already existed routerID and remove the # in front of export inside the "Protocol Kernel" settings. 
To apply the Setting use:
```console
sudo systemctl restart bird
```
Check the status with:
```console
sudo systemctl status bird
```
Check if the OSPF worked:
```console
sudo birdc
BIRD 1.6.3 ready.
bird>

# general bird status
bird> show status

# ospf protocol information
bird> show ospf

# ospf propagated routes
bird> show ospf state

# ospf neighborship
bird> show ospf neighbors
```
#### IP Forwarding
---
To enable IP forwarding we have to edit the sysctl configurations
```console
sudo nano /etc/sysctl.conf
```
Find the commented out line `net.ipv4.ip_forward=1` and enable it for each router.
Then to save and load the new settings run
```console
sysctl -p
```
#### Verification
---
###### Route Failover
To set a specific interface up / down
```console
ip link set dev <interface> [up/down]
```
###### Passive Interface
To check if the client and webserver recieve any OSPF messages use wireshark with the command:
```console
tshark -i <interface> -Y "ospf"
```
###### Access Website
To check if the client can access the webserver use curl with the ip of the webserver's interface and the port it listens to:
```console
curl 192.168.1.100:8080
```
