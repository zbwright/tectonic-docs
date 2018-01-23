<br>
<div class="alert alert-info" role="alert">
    <i class="fa fa-exclamation-triangle"></i><b> Note:</b> This documentation is for an alpha feature. To register for the Tectonic on AWS GovCloud Alpha program, email <a href="mailto:tectonic-alpha-feedback@coreos.com">tectonic-alpha-feedback@coreos.com</a>.
</div>

# Install Tectonic on AWS GovCloud with Terraform

Use this guide to manually install a Tectonic cluster on an [AWS GovCloud][govcloud-account] account.

## Prerequsities

* **CoreOS Account**: Register for a [CoreOS Account][account-login], which provides free access for up to 10 nodes on Tectonic. You must provide the account's License and Pull Secret (available from the account Overview page) during installation.
* **Terraform:** >= v0.10.7 Tectonic Installer includes and requires a specific version of Terraform. This is included in the Tectonic Installer tarball. See the [Tectonic Installer release notes][release-notes] for information about which Terraform versions are compatible.
* [Container Linux Alpha][cl-alpha] contains support for fetching S3 objects from the GovCloud partition, required for Tectonic.
  Set `tectonic_container_linux_channel = alpha` before deploying your cluster.
* **DNS:** Ensure that a PowerDNS server instance is running and reachable from the VPC where the cluster is running.

See [contrib/govcloud](../../../contrib/govcloud) for an example of a prebuilt VPC with restricted VPN access and a PowerDNS server.

## Getting Started

First, clone the Tectonic Installer repository:

```
$ git clone https://github.com/coreos/tectonic-installer.git
$ cd tectonic-installer
```

Initialize Terraform:

```
$ terraform init platforms/govcloud
```

Configure your AWS GovCloud credentials.

```
$ export AWS_ACCESS_KEY_ID=my-id
$ export AWS_SECRET_ACCESS_KEY=secret-key
```

## Customize the deployment

Customizations to the base installation live in `examples/terraform.tfvars.govcloud`. Export a variable that will be your cluster identifier:

```
$ export CLUSTER=my-cluster
```

Create a build directory to hold your customizations and copy the example file into it:

```
$ mkdir -p build/${CLUSTER}
$ cp examples/terraform.tfvars.govcloud build/${CLUSTER}/terraform.tfvars
```

Edit the parameters with your VPC details:

```
tectonic_govcloud_external_vpc_id
tectonic_govcloud_external_master_subnet_ids
tectonic_govcloud_external_worker_subnet_ids
tectonic_govcloud_dns_server_ip
```

## Deploy the cluster

If you are following the [contrib/govcloud](../../../contrib/govcloud) example and deploying from an external machine, connect to the VPN now.

Add the `tectonic_govcloud_dns_server_ip` to your local DNS resolver.

Test out the plan before deploying everything:

```
$ terraform plan -var-file=build/${CLUSTER}/terraform.tfvars platforms/govcloud
```

Next, deploy the cluster:

```
$ terraform apply -var-file=build/${CLUSTER}/terraform.tfvars platforms/govcloud
```

This should run for a little bit, and when complete, your Tectonic cluster should be ready.

### Access the cluster

The Tectonic Console should be up and running after the containers have downloaded. Access it at the DNS name configured in your variables file prefixed by the cluster name. For example: `https://cluster_name.tectonic_base_domain`.

Inside of the /generated folder you should find any credentials, including the CA if generated, and a kubeconfig. Use this to control the cluster with kubectl:

```
$ export KUBECONFIG=generated/auth/kubeconfig
$ kubectl cluster-info
```

### Delete the cluster

```
$ terraform destroy -var-file=build/${CLUSTER}/terraform.tfvars platforms/govcloud
```


[account-login]: https://account.coreos.com/login
[govcloud-account]: http://docs.aws.amazon.com/govcloud-us/latest/UserGuide/govcloud-differences.html
[release-notes]: https://coreos.com/tectonic/releases/
[cl-alpha]: https://coreos.com/releases/#1662.0.0
