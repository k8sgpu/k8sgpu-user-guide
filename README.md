# K8sGPU Overview
K8sGPU is a cloud-native solution to consume remote GPUs in a serverless mode. It is build with [Virtual Kubelet](https://virtual-kubelet.io/) to easly link, access, and consume remote GPUs in your Kubernetes cluster.

![Logo](assets/k8sgpu-architecture.png)

## Use Cases
The solution is ideal for scalable GPU-powered applications, such as ML/IA training, fine tuning, and inference, and other GPU-dependent tasks, while providing a cost-effective and efficient way to connect GPUs additional computational capacity provided by 3dy party remote GPU Cloud Providers.

K8sGPU is ideal for ML/AI teams already using Kubernetes infrastructures who need to increase computing power without migrating pipelines or starting over.  

## How it works

The typical user process begins with ML/IA platform teams installing the k8sGPU agent on their existing Kubernetes Cluster deployed on-prem or any cloud provider. Once deployed, it looks like a real worker node. However, when scheduling a pod on such virtual node, the workload is created on a remote GPU Cloud instead of running in the local cluster. This architecture leverages the scalability and flexibility of Kubernetes and related ecosystem, while integrating seamlessly with remote providers' distributed GPU resources.

## Documentation

* [Quick Start](guides/quickstart.md)
* [Operations](guides/operations.md)
* [Usage from Public Cloud](guides/public-cloud.md)
* [Usage from K3S](guides/k3s.md)
* [Usage from RKE](guides/rke.md)


## FAQ

<details open>
<summary><i>Is the K8sGPU agent free?</i></summary>
Yes, the agent is free of charge. You just pay for the GPU usage.
</details>

<details open>
<summary><i>How do I install the k8sGPU agent?</i></summary>
Install the k8sGPU agent on your Kubernetes cluster using Helm.
</details>

<details open>
<summary><i>Which Kubernetes distributions are supported?</i></summary>
K8sGPU agent runs on most of public Managed Kubernetes, as well as on-premise environments like Kubernetes vanilla and common distributions such as OpenShift, Tanzu, and Rancher. Local environments like k3s and kind are supported;
</details>

<details open>
<summary><i>How do I schedule my pods on the virtual node?</i></summary>
Schedule your pods on the virtual node by setting specific RuntimeClasses. In some cases, affinity rules and tolerations for virtual nodes are required, depending on your Kubernetes environment.
</details>

<details open>
<summary><i>Which container images can I run in the pods?</i></summary>
You can run any container image that is available from any accessible container registry. If you're new to containers, we recommend learning about container technology first.
</details>

<details open>
<summary><i>Can I run multiple pods on the same GPU?</i></summary>
Yes, you can run multiple pods simultaneously on the same GPU, but be aware that GPU memory and processing time are shared among all your workloads.
</details>

<details open>
<summary><i>How do I select the GPU type for my workloads?</i></summary>
K8sGPU supports multiple GPU types. To request a specific GPU, such as the Nvidia A100, refer to the corresponding Kubernetes RuntimeClass resource.
</details>

<details open>
<summary><i>Are the GPUs assigned to my pods shared with other users?</i></summary>
No, each GPU is exclusively assigned to a user and not shared.
</details>

<details open>
<summary><i>Is Multi-Instance GPU (MIG) supported?</i></summary>
MIG support is currently in development and will be available soon.
</details>

<details open>
<summary><i>Are my pods protected from other users?</i></summary>
Yes, your pods are isolated in a multi-tenant environment with strict network policies and are only exposed to the Internet as required.
</details>

<details open>
<summary><i>How can I access my pod's APIs?</i></summary>
Pods on the virtual node run on a remote infrastructure and are not directly accessible within your local cluster. If your pod exposes an API, you can publish it on the Internet. Note that you are the only responsible for securing access to your API.
</details>

<details open>
<summary><i>What if I need private access to my pod's API?</i></summary>
Currently, pod APIs are only exposed to the public Internet as needed. We plan to offer private access via VPN in future updates.
</details>

<details open>
<summary><i>What if my local Kubernetes loses connectivity to the GPU?</i></summary>
Your workloads will continue to run remotely. Once connectivity is restored, your pods will synchronize with their local counterparts.
</details>

<details open>
<summary><i>Where do pods using remote GPUs store their data?</i></summary>
Pods can store and retrieve data from any accessible S3 bucket globally. Support of persistent storage will be released soon.
</details>

<details open>
<summary><i>How do I monitor GPU usage?</i></summary>
Later, we will introduce a dashboard for real-time monitoring and access to historical data.
</details>
