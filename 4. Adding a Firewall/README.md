```console
sudo nft filter forward ip protocol icmp drop
```
```console
sudo nft filter forward iifname "ens3" tcp sport != 8080 drop
```
```console
sudo nft filter forward ip saddr 172.16.0.0/24 accept
```
```console
sudo nft filter forward ip daddr 172.16.0.0/24 accept
```
```console
sudo nft filter forward policy drop
```
```console
sudo nft filter output oifname "ens3" tcp dport 8080 drop
```
```console
sudo nft filter output oifname "ens3" ip protocol icmp drop
```

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
Whenever you edit the nftables.conf (either with commands or directly in the file) you have to restart the service and sometimes reenable it.
```console
sudo systemctl restart nftables
```
```console
sudo systemctl enable nftables
```