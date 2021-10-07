## Commands for Performance
To test performance use iperf with R5 as server and R2 as client. R3 acts as traffic control router. 
#### Creating References
---
Setup Server for iperf on R5<\br>
R5
```console
iperf3 -s
```
Create the reference values: ping with 10 packets and iperf connection witout disturbances
R2
```console
ping 10.0.5.2 [-c 10]
iperf3 -c 10.0.5.2
```
#### Reset Traffic Control
---
After each of the commands below enter
```console
sudo tc qdisc del dev eth0 root
```
to delete the entered settings of traffic control.
#### Delay
---
To add delay to each iperf and ping enter following code to R3:
```console
sudo tc qdisc add dev eth0 root netem delay X ms
```
#### Jitter
---
To add additional Jitter enter the same command used for delay with an additional time that acts as the jitter value.  
R3
```console
sudo tc qdisc add dev eth0 root netem delay X ms Y ms
```
#### Packet Loss
---
To add packet loss to the connection use:
```console
sudo tc qdisc add dev eth0 root netem loss X %
```
#### Bandwidth Limitations
---
In addition to the above commands you can also manipulate the bandwidth for the connection through R3:
```console
tc qdisc add dev eth0 root tbf rate 1mbit burst 32kbit latency 400ms
```