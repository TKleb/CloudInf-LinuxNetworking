## Adding a Firewall to Router 5
#### Table and Chain
---
Nftables' configurations consist of one or more tables with different chains. For this lab you only need the standard ones "input", "forward" and "output". Input handles all packets that have router 5 as destination. Forward handles everything that the router passes through to another destination. Output is used to control the packets that start from router 5. Since only the connection to the webserver has to be altered you only need to modify "forward" and "output".
#### Commands
---
First add a filter for icmp (ping) packets:
```console
sudo nft filter forward ip protocol icmp drop
```
Since nmap has to be disabled aswell drop everything entering "ens3" (iif...) that doesnt originate from the port 8080.
```console
sudo nft filter forward iifname "ens3" tcp sport != 8080 drop
```
The next two commands allow only packets that either have 172.16.0.0/24 as source or destination to go through the router
```console
sudo nft filter forward ip saddr 172.16.0.0/24 accept
```
```console
sudo nft filter forward ip daddr 172.16.0.0/24 accept
```
With adding "policy drop" you tell the router to drop everything that wasn't allowed in an above rule.
```console
sudo nft filter forward policy drop
```
"ens3" is the output interface for this router (oif...) the command to disable curl by not allowing tcp connection to port 8080 looks like this: 
```console
sudo nft filter output oifname "ens3" tcp dport 8080 drop
```
Since the router 5 should still be pingable from other routers icmp is only dropped when its going out from "ens3".
```console
sudo nft filter output oifname "ens3" ip protocol icmp drop
```
#### Show the configurations
---
At the end the use one of the below commands to show the firewall table:
```console
sudo nft list table filter
```
```console
sudo nano /etc/nftables.conf
```
It will look like this
```console
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
        chain input {
              type filter hook input priority 0;
        }

        chain forward {
              type filter hook forward priority 0;
              ip protocol icmp drop;
              iifname "ens3" tcp sport != 8080 drop;
              ip saddr 172.16.0.0/24 accept;
              ip daddr 172.16.0.0/24 accept;
              policy drop;
        }     

        chain output {
              type filter hook output priority 0;
              oifname "ens3" tcp dport 8080 drop;
              oifname "ens3" ip protocol icmp drop;
        }
}
```
The connection works this way as intended but to make the firewall work stateful you have to change the forward chain to look like this:
```console
         chain forward {
              type filter hook forward priority 0; policy drop;
              ip protocol icmp drop;
              iifname "ens3" tcp sport != 8080 drop;
              ct state established, related accept;
              iifname "ens2" ip saddr 172.16.0.0/24 ct state accept;
         }    
```
#### Start nftables and run the Firewall
---
Whenever you edit the nftables.conf (either with commands or directly in the file) you have to restart the service and sometimes reenable it.
```console
sudo systemctl restart nftables
```
```console
sudo systemctl enable nftables
```
#### Testing Firewall
---
The firewall needs to be tested for pings, nmap and curl.  
First try pinging the webserver from the client and the routers. No one should be able to bring a ping through:
```console
ping 192.168.1.100
```
Nmap should not be able to see any open port. To make nmap scan a specific portrange use the -p command:
```console
sudo nmap -p 1-65535 192.168.1.100
```
Finally test for curl. Only the client should be able to connect to the webserver with it:
```console
curl 192.168.1.100:8080
```