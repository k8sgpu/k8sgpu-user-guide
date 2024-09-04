# K8sGPU Operations

Once the K8sGPU agent is installed, the virtual node acts as a proxy to remote GPU resources hosted by the GPU provider. You can schedule and run pods as if they were on a local node in your cluster.


## RuntimeClasses
Depending on the GPU provider of choice, one or more RuntimeClasses are assigned to the K8sGPU agent:

```
$ kubectl get runtimeclasses
NAME                  HANDLER AGE
seeweb-nvidia-2xa6000 nvidia  13m
seeweb-nvidia-1xa6000 nvidia  13m
seeweb-nvidia-1xa100  nvidia  13m
```

Use the RuntimeClass to request a specific GPU model.

For example, to request a GPU NVIDIA-RTX-A6000, use the appropriate RuntimeClass in your pod manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nvidia-smi
spec:
  restartPolicy: OnFailure
  runtimeClassName: seeweb-nvidia-1xa6000 # <<<< use NVIDIA-RTX-A6000
  containers:
  - name: nvidia
    image: nvidia/cuda:12.6.0-base-ubuntu20.04
    command: ["/bin/bash", "-c", "--"]
    args: ["sleep 3600"]
    imagePullPolicy: Always
    resources:
      limits:
        nvidia.com/gpu: 1
```

By inspecting the RuntimeClass annotations, you can see the usable pod resources:

```json
$ kubectl get runtimeclass seeweb-nvidia-1xa6000 -o jsonpath='{.metadata.annotations}' | jq

{
  "limits.k8s.gpu/cpu": "8000m",
  "limits.k8s.gpu/default": "1",
  "limits.k8s.gpu/memory": "32861620Ki"
}
```

So the pod using this RuntimeClass cannot request more than:

- GPU: `1xNVIDIA-RTX-A6000`
- CPU: `8xCPUs`
- Memory: `32861620Ki`

If you do not specify resources in the pod manifest, the K8sGPU agent will assign them automatically as configured in the RuntimeClass. If you specify more resources than allowed by the RuntimeClass, the pod will not run and will remain pending.  

For example, the following pod will run:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nvidia-smi
spec:
  restartPolicy: OnFailure
  runtimeClassName: seeweb-nvidia-1xa6000
  containers:
  - name: nvidia
    image: nvidia/cuda:12.6.0-base-ubuntu20.04
    command: ["/bin/bash", "-c", "--"]
    args: ["sleep 3600"]
    imagePullPolicy: Always
    resources:
      limits:
        nvidia.com/gpu: 1
        memory: 16Gi
        cpu: 2
```

While this other pod will remain pending:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nvidia-smi-unschedulable
spec:
  restartPolicy: OnFailure
  runtimeClassName: seeweb-nvidia-1xa6000
  containers:
  - name: nvidia
    image: nvidia/cuda:12.6.0-base-ubuntu20.04
    command: ["/bin/bash", "-c", "--"]
    args: ["sleep 3600"]
    imagePullPolicy: Always
    resources:
      limits:
        nvidia.com/gpu: 2
        memory: 64Gi
        cpu: 1
```

This pod will be scheduled as the total request matches with the RuntimeClass:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nvidia-smi-double
spec:
  restartPolicy: OnFailure
  runtimeClassName: seeweb-nvidia-2xa6000 # <<<< use 2xNVIDIA-RTX-A6000
  containers:
  - name: nvidia-one
    image: nvidia/cuda:12.6.0-base-ubuntu20.04
    command: ["/bin/bash", "-c", "--"]
    args: ["sleep 3600"]
    imagePullPolicy: Always
    resources:
      limits:
        nvidia.com/gpu: 1
        memory: 16Gi
        cpu: 1
  - name: nvidia-two
    image: nvidia/cuda:12.6.0-base-ubuntu20.04
    command: ["/bin/bash", "-c", "--"]
    args: ["sleep 3600"]
    imagePullPolicy: Always
    resources:
      limits:
        nvidia.com/gpu: 1
        memory: 16Gi
        cpu: 1
```

Make sure your pod resources are aligned with the RuntimeClass; in case of doubts, leave it empty as the K8sGPU agent will do the job.

## Virtual Node Capacity and Allocatable

Depending on the GPU provider's capacity, the virtual node reflects the overall capacity for your RuntimeClasses. Check the capacity and the allocatable resources for your virtual node:

```json
kubectl get node k8s.gpu -o jsonpath='{.status.capacity}{.status.allocatable}' | jq
{
  "cpu": "63",
  "memory": "263567056Ki",
  "nvidia.com/gpu": "8",
  "pods": "8"
}
{
  "cpu": "63",
  "memory": "263567056Ki",
  "nvidia.com/gpu": "8",
  "pods": "8"
}
```

As you allocate resources for your pods, the virtual node's allocatable resources are decremented accordingly. For example, if you allocate the following pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nvidia-smi
spec:
  restartPolicy: OnFailure
  runtimeClassName: seeweb-nvidia-1xa6000
  containers:
  - name: nvidia
    image: nvidia/cuda:12.6.0-base-ubuntu20.04
    command: ["/bin/bash", "-c", "--"]
    args: ["sleep 3600"]
    imagePullPolicy: Always
    resources:
      limits:
        nvidia.com/gpu: 1
        memory: 16Gi
        cpu: 2
```

Then the virtual node's allocatable resources become:


```json
kubectl get node k8s.gpu -o jsonpath='{.status.capacity}{.status.allocatable}' | jq

{
  "cpu": "63",
  "memory": "263567056Ki",
  "nvidia.com/gpu": "8",
  "pods": "8"
}
{
  "cpu": "61",
  "memory": "246789840Ki",
  "nvidia.com/gpu": "7",
  "pods": "7"
}
```

Once you allocate all the capacity, the virtual node's allocatable resources will reflect accordingly:

```json
kubectl get node k8s.gpu -o jsonpath='{.status.capacity}{.status.allocatable}' | jq

{
  "cpu": "63",
  "memory": "263567056Ki",
  "nvidia.com/gpu": "8",
  "pods": "8"
}
{
  "cpu": "2",
  "memory": "360456Ki",
  "nvidia.com/gpu": "0",
  "pods": "0"
}
```

## Storage

Pods running on virtual node can use ConfigMap and Secrets as regular Kubernetes pods.

> Note: Support of Persistent Volumes is going to be released soon.  

## Access to Applliations

When enabled by the remote GPUs Service Provider, pods running on the GPU can be accessed only from the public Internet by creating a Kubernetes Service in your local cluster with `type=LoadBalancer` and `loadBalancerClass=k8s.gpu` as in the following example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: inference
spec:
  loadBalancerClass: k8s.gpu # Maake sure to use this loadBalancerClass.
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    run: inference  # This selector must match the Pod label
  sessionAffinity: None
  type: LoadBalancer
```

> Make sure in your cluster, there are no other Load Balancer Controllers, eg. MetalLB, listening for the same `loadBalancerClass=k8s.gpu`. Any default load balancer implementation from Public Cloud Providers should ignore Services that set this field.

The HTTPS endpoint is in the form of `${service.metadata.uid}-${tenant.metadata.name}.cloud.provider.tld` where `cloud.provider.tld` is the FQDN of the remote GPU Service Provider.

Once the service is created, it should appears as this:

```
$ kubectl get svc
NAME         TYPE           CLUSTER-IP    EXTERNAL-IP                                                PORT(S)        AGE
inference    LoadBalancer   10.32.1.166   8d0a8f33-e9f9-434a-b3de-9ae58ac9de76-skg00000.cloud.provider.tld   80:31466/TCP   3m
```

You can access the pod from public internet:

```
$ curl -k https://8d0a8f33-e9f9-434a-b3de-9ae58ac9de76-skg00000.cloud.provider.tld
It works!
```

> If you want do that, make sure you have your authentication in place!

## Pod Restrictions

The K8sGPU is a work-in-progress solution. As with many new technologies, it can be improved over time. Currently, the following limitations apply:

- Pods cannot mount local storage
- Pods cannot access other local services running in your cluster 
- Pods cannot be exposed on the local cluster
- Pods cannot violate the [Pod Security Standard Baseline ](https://kubernetes.io/docs/concepts/security/pod-security-standards/) profile. 


## Troubleshooting

Pods running on the virtual node can be inspected as usual:

```
$ kubectl describe pod stable-diffusion-pod
```

Logs are also collected:

```
$ kubectl logs -f stable-diffusion-pod
```

It is possible to exec into the pod:

```
$ kubectl exec -it stable-diffusion-pod -- bash
```

If a pod is not scheduled, make sure you have set the resource requests and limits according to the assigned RuntimeClass.



