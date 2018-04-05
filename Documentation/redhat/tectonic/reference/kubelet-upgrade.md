# Upgrading the Kubelet manually

Tectonic installs the Kubelet as part of the infrastructure provisioning install step, which typically uses a cloud autoscaling group or other provisioning tool to specify where to find the Kubelet image and the desired version.

To upgrade the Kubelet manually, this version string can be updated by hand.

## Find the desired version

When updating the Kubelet, first upgrade the cluster to the latest available version. Afterwards, find the current version of the cluster in order to match the Kubelet versions with it.

```
$ kubectl --namespace=tectonic-system get configmap/tectonic-config -o jsonpath="{..data.tectonicVersion}"
1.8.4-tectonic.3
```

On the above cluster, pay attention to the `1.8.4` part only, and cross reference that with the [available versions on Quay.io][hyperkube], like `v1.8.4_coreos.0`.

[hyperkube]: https://quay.io/repository/coreos/hyperkube?tag=latest&tab=tags

## Upgrade each Node

Upgrade each Node in the cluster one by one via SSH. In most cases, the worker Nodes will need to be accessed through a bastion host.

Use kubectl or the Console to obtain a list of IP addresses for each Node. For each, run these commands:

1. Update `/etc/kubernetes/kubelet.env` to match the version of the cluster
2. Restart the Kubelet with `sudo systemctl restart kubelet.service`

To verify you have updated successfully, view the Node's detail page in the Console or use kubectl:

```
$ kubectl get nodes -o wide
```
