# Install Helm and kubectl on student laptop

## Step 1 - Installing Helm on student laptop

Follow the instructions here to install helm on your student laptop

- [https://github.com/helm/helm#install](https://github.com/helm/helm#install)

## Step 2 - Install kubectl on student laptop

Follow the instructions here to install `kubectl` with chocolatey

- [https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/#install-on-windows-using-chocolatey-or-scoopl](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/#install-on-windows-using-chocolatey-or-scoop)
- Steps 3-6 will be done as your student user on you laptop **NOT ADMINISTRATOR** (Use your gitbash terminal in vscode)
- Step 6 tells you to configure kubectl to use a remote cluster
  - Each time you deploy kubernetes a cluster with RKE2, the contents for this config file can be found in `/etc/rancher/rke2/rke2.yaml`
  - Copy the contents of this file and change the server IP address from `https://127.0.0.1:6443` to `https://<server-ip>:6443`
  - **YOU WILL NEED TO FIND THE NEW CONTENTS OF THIS FILE EACH TIME YOU REDEPLOY RKE2**

Verify kubectl using the instructions here using the config file from your rke2-cluster

- [https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/#verify-kubectl-configuration](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/#verify-kubectl-configuration)
- You can now manage your kubernetes cluster from your dev laptop.
  - View your node and the pods running in your cluster with `kubectl get nodes` and `kubectl get pods -A`

```