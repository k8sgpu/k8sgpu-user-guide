K3s agents and servers maintain websocket tunnels between nodes that are used to encapsulate bidirectional communication between the control-plane (apiserver)

Edit k3s service ```/etc/systemd/system/k3s.service``` and add ```'--egress-selector-mode=disabled``` to disable the tunnel encapsulation, then you can redirect the flow to cloudcore.

```
ExecStart=/usr/local/bin/k3s \
       server \
        '--egress-selector-mode=disabled' \
```
