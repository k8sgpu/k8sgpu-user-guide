# Enabling Cluster Signing in RKE (Rancher Kubernetes Engine)

When running **Rancher 2.4.5** with **Kubernetes 1.18.10**, you may encounter issues related to certificate signing by the `kube-controller-manager`.  
This typically happens because the controller-manager is missing the required certificate and key used for signing.

---

## 🛠 Problem

The Kubernetes controller-manager requires access to the **cluster signing certificate and key** in order to sign new certificates (e.g., for kubelets).  
Without these files, certificate operations may fail or be incomplete.

---

## ✅ Solution

To fix this, you must add the appropriate certificate and key paths to the `kube-controller` section in your `cluster.yml` file using the `extra_args` directive.

> 🔥 **Important:** Be sure to place this in the correct `kube-controller` section!  
> Other components (like `kubelet`) also support `extra_args`, and placing these values in the wrong section will not work and may lead to time-consuming troubleshooting.

---

## 🧩 Patch Example

Here’s the patch you need to apply in your `cluster.yml`:

```yaml
kube-controller:
  cluster_cidr: 10.233.64.0/18
  service_cluster_ip_range: 10.233.0.0/18
  extra_args:
    cluster-signing-cert-file: /etc/kubernetes/ssl/kube-ca.pem
    cluster-signing-key-file: /etc/kubernetes/ssl/kube-ca-key.pem

scheduler:
kubelet:
  cluster_domain: cluster.local
```

---

## 📝 Explanation of Parameters

- **`cluster-signing-cert-file`**: Path to the cluster CA certificate (`kube-ca.pem`)
- **`cluster-signing-key-file`**: Path to the cluster CA key (`kube-ca-key.pem`)

These parameters tell the `kube-controller-manager` which certificate authority to use when signing new kubelet certificates.

---

## 💡 Additional Tips

- ✅ Double-check the **indentation** and placement of the `extra_args` block.
- 🔁 After making changes, **redeploy the cluster** using `rke up`.
- 🛑 Always **backup your existing `cluster.yml`** before making modifications.

---

## 📎 References

This issue is discussed in detail in [RKE GitHub Issue #546](https://github.com/rancher/rke/issues/546).
