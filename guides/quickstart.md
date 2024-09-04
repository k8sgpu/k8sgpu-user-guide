# Quickstart

You need for your `<tenant_id>` number and your credentials, i.e. a `kubeconfig` file
`<tenant_id>.kubeconfig` to access the remote GPUs infrastructure.

Currently Helm is used to install the agent. Make sure you have `helm` installed in your workstation and you have admin permissions on your cluster.

1. Install the Chart:

```
$ helm repo add clastix https://clastix.github.io/charts
$ helm repo update

$ helm upgrade --install k8sgpu clastix/k8sgpu \
    --namespace kube-system \
    --set "k8sgpuOptions.tenantName=your_tenant_name" \
    --set-file "kubeConfigSecret.content=/path/to/your/gpu-remote-cluster/kubeconfig"
```

2. You will end up with a virtual node in your kubernetes cluster:

```
$ kubectl get nodes
NAME    STATUS ROLES AGE VERSION
k8s.gpu Ready  agent 37s v1.2.2
...
```

3. Check your assigned `RuntimeClasses`:

```
$ kubectl get runtimeclasses
NAME HANDLER AGE
seeweb-nvidia-2xa6000 nvidia 13m
```

4. Usage

Now you can schedule your workloads on the virtual node:

```yaml
$ cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nvidia-smi
spec:
  restartPolicy: OnFailure
  runtimeClassName: seeweb-nvidia-2xa6000
  containers:
  - name: nvidia
    image: nvidia/cuda:12.6.0-base-ubuntu20.04
    command: ["/bin/bash", "-c", "--"]
    args: ["sleep 3600"]
    imagePullPolicy: Always
EOF
```

The pod is now running on the virtual node and can access the assigned GPU specified by the RuntimeClass:

```
$ kubectl get pods
NAME        READY   STATUS    RESTARTS      AGE
nvidia-smi  1/1     Running   0             37s

$ kubectl exec nvidia-smi -- nvidia-smi

+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.54.15              Driver Version: 550.54.15      CUDA Version: 12.6     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA RTX A6000               On  |   00000000:06:00.0 Off |                  Off |
| 30%   34C    P8             31W /  300W |       1MiB /  49140MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
|   1  NVIDIA RTX A6000               On  |   00000000:07:00.0 Off |                  Off |
| 30%   39C    P8             34W /  300W |       1MiB /  49140MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
                                                                                         
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+

```

3. Uninstall the Chart

Delete all pods running on the virtual node and remove the chart using Helm:
   
```
$ helm uninstall k8sgpu --namespace kube-system 
```

The virtual node is then removed from your kubernetes cluster.