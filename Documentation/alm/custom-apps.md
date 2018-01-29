# Working with custom applications

Tectonic provides the means to enable and deploy custom applications.

Use Tectonic Console to enable custom applications for a selected namespace, then initialize and configure the service for use within that namespace.

## Enabling custom applications

Tectonic admins deploy applications on selected namespaces, granting users with access to those namespaces permission to initialize and deploy the service. For more granular control, create custom RBAC roles and permissions to control access to these services within defined namespaces.

Cluster admins may select the teams and namespaces for whom applications will be enabled.

Once enabled in a namespace, normal Kube roles and bindings can be used to further control access to edit or delete these resources.

### Using Tectonic Console

Create button will not be available in Lithium.

### Using kubectl

To enable applications using kubectl, create an `InstallPlan-v1` resource in the desired namespace.

For example:

```yaml
apiVersion: app.coreos.com/v1alpha1
kind: InstallPlan-v1
metadata:
   namespace: default
   name: etcd-installplan
spec:
   clusterServiceVersionNames:
   - etcdoperator.v0.7.0
   approval: Automatic
```

## Creating Instances

Custom applications may be deployed into any namespace for which they are enabled.

1. Go to *Applications > Available Applications* and select the namespace for the service you wish to deploy.
2. Click *Create New* to open a YAML manifest template for the instance.
3. Edit the manifest to rename the app, and customize it if necessary, and click *Create*.

A new instance will be deployed into the selected namespace. Once created, Console displays the following information for each instance:

* The *Overview* tab provides detailed information on the app, including version number and the namespace(s) into which the app is deployed. This page also displays graphs which show CPU, Memory, and Filesystem Usage for the app, and the status of its members (Pods).
* The *YAML* tab opens a page in which the custom resource definition may be reviewed for the selected app.
* The *Resources tab* lists and itemizes all related resources for the selected cluster instance, including number of deployed Pods, services, and backups (available options vary, specific to each service).

## Updating applications

Trigger a rolling update for an application by clicking down into an instance’s Details page, and editing its YAML manifest.
