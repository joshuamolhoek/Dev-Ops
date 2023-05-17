# Creating a helm chart manifest

Even though we do not install helm on our RHEL 8 servers we run Kubernetes on, helm is integrated into the RKE2 cluster.  We will not be able to run helm commands like we did on the student laptops without installing the helm binary on the system, but we can use the integrated helm another way.

We can create a HelmChart resources the same way we created Kubernetes resources using manifest files for a deployment, namespace, and service.

## Step 1 - Log into your student laptop

## Step 2 - package the helm chart

Before we can use the helm chart in a manifest file, we must package it on our dev laptop

navigate to the helm chart directory we use in the previous lab:

```shell
cd ~/helm-charts
```

list the files in the directory.

You should see the apache-stud<student-number> directory

```shell
ll
```

Using helm, package the helm chart

```shell
helm package apache-stud<student-number>/
```

List the files in your directory again

You should now see a tgz file with the name and version of your app

```shell
ll
```

Move this file to your admin node

```shell
scp <tgz-file> <username>@<admin-server-IP>:/home/<username>
```

## Step 3 - Log into the Admin node

## Step 4 - Convert to base64

RKE2 will want to read the helm-chart in base64 so we must convert it before we can use it.

make sure the file you just transferred in the previous step exists in your home directory on the admin node

```shell
ls -al
```

Convert to base64

- This command will:
  - `cat` the file
  - use the output of `cat` to convert it to `base64`
    - the `-w 0` option tells base64 not to wrap the lines
  - redirect the output of `base64` to a text file we can use later

```shell
cat <tgz-filename> | base64 -w 0 > apache-stud<student-number>-base64.txt
```

## Step 5 - Create the helm chart manifest file

Create and open a file for editing to use as the HelmChart manifest:

```shell
vim apache-stud<student-number>-chart.yaml
```

Add the following crds to the file:

**FILL IN THE NAME AND NAMESPACE FOR YOUR APACHE APPLICATION**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: <namespace>
---
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: <app-name>
  namespace: <namespace>
spec:
  chart: 
  targetNamespace: <namespace>
  valuesContent:
  chartContent:
```

Save and close the file

We are now going to place the base64 of our tgz file into the chartContent of the file.

**BE SURE TO USE THE DOUBLE ">" (>>) OR IT WILL OVERWRITE THE WHOLE FILE**

```shell
cat apache-stud<student-number>-base64.txt >> apache-stud<student-number>-chart.yaml
```

Open the file and remove the `newline` after chart content so it looks like this:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: <namespace>
---
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: <app-name>
  namespace: <namespace>
spec:
  chart: 
  targetNamespace: <namespace>
  valuesContent:
  chartContent: H4sIFAAAAAAA/ykAK2FIUjBjSE02THk5NWIzVjBkUzVpWlM5Nk9WVjZNV2xqYW5keVRRbz1IZWxtAOy963rbNrow2t+6Cixl5kmbZcrHJDP+dte3XNttvSZx9FhO8qzV6ZdAJCShpgAOANpWO/2uZV/LvrL94ESCZ1KSnaRD/kgsEocXL168ZwAhJfMFZWT3dAGZGK3gMvxq28/e3t7ei6Mj9f/e3l7+//3nhy++2j86eH5weHR0+PLlV3v7h3uHL78Ce1uHpOSJuYDsq72N+8oP7gt5YITfIcYxJcfgdn8Ao8j5OToc7Q0CxH2GI6HevTLkAjAHEASYC4ansUABmIbUvwFcUAbnCPAVF2gJZpSBv8VTxAgSiI8GC7pEx2AhRMSPd3fnWCzi6ciny92EDO0fA+zL/mxRBu9GunjMEfMpEYgIVdMn/mwXMnFH2c3uEnKB2G7E
---snippet---
```

Copy the apache-stud<student-number>-chart.yaml file to Server 1

```shell
scp apache-stud<student-number>-chart.yaml <username>@<server-1-IP>:/home/<username>/
```

## Step 6 - Log in to Server 1

## Step 7 - Deploy the application

Copy the file to the manifests directory to deploy the application:

```shell
cp apache-stud<student-number>-chart.yaml /var/lib/rancher/rke2/server/manifests/
```

You should now be able to see your application resources deployed in the namespace just like we did when we deployed it with helm from our laptops.

## Step 8 - Delete the helm-chart

To delete the HelmChart deployed by RKE2, there are 4 things we need to do.

- Remove the manifest file from the manifests directory
- Delete the addon created by RKE2
- Delete the HelmChart
- Delete the namespace

1. Remove the manifest file from the manifests directory
Move to the manifests directory

```shell
cd /var/lib/rancher/rke2/server/manifests/
```

list the files and find the manifest file you put there

```shell
ll
```

Remove the file

```shell
rm <filename.yaml>
```

2. Remove the addons
RKE2 Creates addons to track the files in the manifests directory

To view them from the command line

```shell
kubectl get addons -A
```

Find the addon created for your apache application

Remove the addon

```shell
kubectl delete addons -n kube-system <addon-name>
```

3. Delete the helmchart
Helm charts deployed by RKE2 can be seen from the command line

```shell
kubectl get helmcharts -A
```

Find your helm chart in this list and delete it

```shell
kubectl delete helmchart -n <namespace> <helm-chart-name>
```

4. Delete the namespace
Since the namespace was created in the manifest file, but outside of the HelmChart we still need to delete the namespace separately

First we want to check to make sure there are no resources left in the namespace.  Deleting a namespace may hang up and cause problems if resources still exist in the namespace.

```shell
kubectl get all -n apache-stud<student-number>
```

```shell
kubectl delete namespace apache-stud<student-number>
```

## Step 9 - Verify the removal of the helm chart

All of the resources we previously created should now be gone from the cluster including the helmchart, namespace, deployment, and service

```