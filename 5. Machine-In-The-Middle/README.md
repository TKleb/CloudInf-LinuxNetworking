## Machine In The Middle
#### Interrupting Response and Altering It
---
First you need to disable the Firewall by running
```console
sudo systemctl stop nftables
```
on router 5. This needs to be done to alter the response from the webserver since only traffic originating from the clients network can go through the firewall.
Start the proxy with 
```console
mitmproxy
```
Send the traffic that you want to intercept by entering following code on the client device:
```console
sudo curl --proxy 10.0.255.2:8080 192.168.1.100:8080
```
Then accept the traffic by entering "a". Then change the Tab to "Response". Enter "e" to edit the response and since the editor is vim exit by pressing ":wq" followed by "a" again to let the reponse go to the client.

#### Interrupting Request And Sending Fake Response
First you need to create a python script which takes the message if it request if its destination is the webserver and sends back a custom response (In this case the same response you would get from the server but you can enter whatever you want).

```python
from mitmproxy import http


def request(flow: http.HTTPFlow) -> None:
    if "192.168.1.100:8080" in flow.request.url:
        flow.response = http.Response.make(
            200,  # (optional) status code
            '<h1>Hello Python Flask!</h1><hr> <p>A simple hello web app, in a docker image, using debian, python and flask!</p><p>Hostname: python-flask-hello</p><p>Requests: 18<br \></p><p>Network Interface: eth0<br \>--> IP Address: 192.168.1.100<br \>--> Netmask: 255.255.255.0<br \></p>',  # (optional) content
            {"Content-Type": "text/html"}  # (optional) headers
        )
```
Save this file on the MITM machine by using 
```console
touch my_script.py
```
and then copy paste the above python code into the file by opening it with nano.
To then run the script use
```console
mitmdump -s my_script.py
```
Reenter the curl command using MITM as proxy and you get the response you entered.