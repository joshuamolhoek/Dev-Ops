
# Connecting the cluster to a private repository

#

## Step 1 - Revert server 1 to a snapshot

#

## Step 2 - Install RKE2

Use the script you built you have already built and tested to install RKE2

#

## Step 3 - Build `registries.yaml`

#

At this point you installed a single node rke2 server cluster on RHEL8.  After this lab we will need images from repositories outside of the cluster.  

#

RKE2 can connect to a private repository on start up as long as a file named `registries.yaml` exists and is **configured correctly** in the `/etc/rancher/rke2` directory. Additional information not in this documentation can be found [here](https://docs.rke2.io/install/containerd_registry_configuration/)

#

Create and open the `registries.yaml` file for editing

```bash
vim /etc/rancher/rke2/registries.yaml
```

Fill in the required information for you cluster and add the following to the file:

```yaml
mirrors:
  docker.io:
    endpoint:
      - "<harbor_hostname>.vip.<domain>"
  quay.io:
    endpoint:
      - "<harbor_hostname>.vip.<domain>"
  k8s.gcr.io:
    endpoint:
      - "<harbor_hostname>.vip.<domain>"
  "<harbor_hostname>.vip.<domain>":
    endpoint:
      - "<harbor_hostname>.vip.<domain>"
configs:
  "<harbor_hostname>.vip.<domain>":
    tls:
      insecure_skip_verify: true
```

What is being done here?

- mirrors such as `docker.io`, `quay.io`, etc. are aliases for public repositories to tell the RKE2 to pull from Harbor instead of the public repository.
- In configs, we are skipping the verification of the tls certificate

## Step 4 - Restart the `rke2-server` service

For these changes to take effect, the `rke2-server` service needs to be restarted.

> **NOTE** If there are multiple nodes in the cluster the **`registries.yaml` will need to be deployed on every node in the cluster.**

```bash
systemctl restart rke2-server
```

## Step 5 - Pull a test image

A test image named `library/busybox:private-repo-test`has been loaded into harbor. Deploy a pod with a container using this image to ensure the cluster is pulling from the Harbor repository.

Create a manifest file to deploy a the busybox pod.

```bash
vim busybox-test.yaml
```

Add the following to the contents of the file.

**Change the image name and tag** to `busybox:private-repo-test`

We are telling busybox to print a statement with the `echo` command.  We will be able to view this in the logs once the pod has deployed.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: <image-name:image-tag>
    command:
      - echo
      - "busybox successfully deployed"
    imagePullPolicy: Always
    name: busybox
  restartPolicy: OnFailure
  ```

Apply the manifest file:

```bash
kubectl apply -f busybox-test.yaml
```

Make sure the pod was created.  The status should show complete.

**NOTE** This pod was deployed in the default namespace so we do not have to specify the namespace.

```bash
kubectl get pods
```

Check the logs of the pod to see the output of the command we had it run in the manifest file.

```bash
kubectl logs busybox
```
```