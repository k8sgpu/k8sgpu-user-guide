# Usage in Public Cloud Providers

## K8sGPU on AWS EKS

Prior to run K8sGPU on AWS EKS, some changes must be addressed.

### Using the AWS Access Entries API

Please, refer to the official [AWS documentation](https://docs.aws.amazon.com/eks/latest/userguide/access-entries.html) and use your preferred method (AWS CLI or AWS UI).

For example, with the AWS CLI:

```
aws eks create-access-entry \
--cluster-name my-cluster \
--principal-arn arn:aws:iam::111122223333:role/EKS-my-cluster-self-managed-ng-1 \
--type EC2_Linux \
--username system:node:k8s.gpu
```

From the UI, a new entry for the K8sGPU node must be created to the desired username, such as `system:node:k8s.gpu`.

### Updating the `aws-auth` ConfigMap

The `aws-auth` Config Map contains the entitlements for entities allowed to interact with the EKS Control Plane.

```yaml
# kubectl -n kube-system get cm aws-auth -o yaml
apiVersion: v1
data:
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::<IAM_ID>:role/<ROLE_NAME>
      username: system:node:{{EC2PrivateDNSName}}
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::<IAM_ID>:role/<ROLE_NAME>
      username: system:node:k8s.gpu
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
```

Please note in the section `mapRoles` the new entry added matching the username `system:node:k8s.gpu`. This entry allows the `k8s.gpu` node to be recognized by the AWS Cloud Controller Manager, preventing its deletion.

> The assigned AWS Role Name to the K8sGPU instance can match the regular ones.
> For security reasons, an empty role can be used.

### Overriding the CertificateSigningRequest signer name

The CLI option `--csr-signer` must be set to `aws` if K8sGPU is delegating the Kubelet certificates to the `CertificateSigningRequest` API.

## K8sGPU on Azure AKS
On Azure AKS, the virtual node is automatically tainted as no CNI is deployed.

A toleration is required to schedule a pod on the virtual node:

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
  tolerations:
  - key: "node.kubernetes.io/network-unavailable"
    operator: "Exists"
    effect: "NoSchedule"
```
