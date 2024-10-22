K3s agents and servers maintain websocket tunnels between nodes that are used to encapsulate bidirectional communication between the control-plane (apiserver)

Edit ```/etc/rancher/k3s/config.yaml``` and add ```egress-selector-mode: disabled```  to disable the tunnel encapsulation, then you can redirect the flow to cloudcore.

Alternative: create override conf for systemd unit

Create  ```/etc/systemd/system/k3s.service.d/override.conf``` 

```
[Service]
ExecStart=
ExecStart=/usr/local/bin/k3s server --egress-selector-mode=disabled
```
reload conf and restart:

```
sudo systemctl daemon-reload
sudo systemctl restart k3s
```
