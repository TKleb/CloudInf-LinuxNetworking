## Commands for Performance
To test performance use iperf with R5 as server and R2 as client. R3 acts as traffic control router. 
#### Creating References
---
Setup Server for iperf on R5  
R5
```console
iperf3 -s
```
Create the reference values: ping with 10 packets and iperf tcp or udp (-u for udp and -b to enter bandwidth, in this case 100 Mbit)witout disturbances  
R2
```console
ping 10.0.5.2 [-c 10]
iperf3 -c 10.0.5.2
iperf3 -c 10.0.5.2 -u -b 100m
```
#### Reset Traffic Control
---
After each of the commands below enter
```console
sudo tc qdisc del dev ens4 root
```
to delete the entered settings of traffic control.
#### Delay
---
To add X delay to each iperf and ping enter following code to R3:
```console
sudo tc qdisc add dev ens4 root netem delay Xms
```
#### Jitter
---
To add Y additional Jitter enter the same command used for delay with an additional time that acts as the jitter value.  
R3
```console
sudo tc qdisc add dev ens4 root netem delay Xms Yms
```
#### Packet Loss
---
To add X amount of packet loss to the connection use:
```console
sudo tc qdisc add dev ens4 root netem loss X%
```