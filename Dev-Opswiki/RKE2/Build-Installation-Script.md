# Build Installation Script
> follow the steps for building your script to automate the install

# Automate the installation of RKE2

## Step 1 - Log into your student admin VM

The following steps will be completed on your admin VM

## Step 2 - Checking kubernetes resources with kubectl

In the manual installation, we could run the command `kubectl get pods -n kube-system` to check on the installation of RKE2 and make sure the pods are running.

We will want to write it into our script to check for us before it continues on to the next process or echos the completion status of the task.

#

kubectl is a powerful tool and there are commands for almost everything you would need to accomplish.

To check the status of a deployment, statefulset, or daemonset, you can run the following command:

```bash
kubectl -n <namespace> rollout status <container_type(deployment, statefulset, or daemonset)>.apps/<app_name>

example:
kubectl rollout status -n kube-system deployment.apps/rke2-coredns-rke2-coredns
```

This will return the rollout status of the deployment:

```bash
deployment "rke2-coredns-rke2-coredns" successfully rolled out
```

There is a problem with writing the command into our script though.  If the deployment is not yet deployed, the command will return a `non-zero` exit code and the script will stop.

We want to wait for the container to become ready before moving on without the script failing so we can create a `while loop`

We can use the `!` operator to only run the while loop when the value of the command returns a non-zero exit code

```bash
while ! kubectl -n <namespace> rollout status <container_type>.apps/<app_name>; do
    sleep 2 && echo "Waiting for <app-name> to come up"
done
```

Instead of writing this loop out every time we want to use it, we will create a function we can pass variables into:

```shell
check_container_status{
    while ! kubectl -n ${namespace} rollout status ${container_type}.apps/${app_name}; do
        sleep 2 && echo "Waiting for <app-name> to come up"
    done
}
```

This can be used in our **install_rke2** function by defining the variables when we call the **check_container_status** function

EXAMPLE:

```shell
app_name="rke2-coredns-rke2-coredns" namespace="kube-system" container_type="deployment" check_container_status
```

## Step 2 - Build the `install_rke2` function

In this lab you will automate the installation of RKE2

Your script must contain the following:

- a function named `install_rke2` contained in a file named `install_rke2.sh` in the source directory.
- Create a function named `check_container_status` in a file named `check_container_status.sh` in the source directory.
- Every action written into the script will have a comment explanation before the action
- Every action written into the script will print the succesfull completion of the action using `echo`
- The function `install_rke2` should contain the following actions:
  - Create the `config.yaml` with variables for the IPs and hostnames
  - run the install.sh script
  - enable and start the rke2-server.service
  - Call on the `check_container_status` function to perform a check on the following resources in the `kube-system` namespace:
    - daemonset.apps/rke2-canal
    - daemonset.apps/rke2-ingress-nginx-controller
    - deployment.apps/rke2-coredns-rke2-coredns
    - deployment.apps/rke2-coredns-rke2-coredns-autoscaler
    - deployment.apps/rke2-metrics-server
  - echo completion of the function


# Your install_rke2.sh will look like this
```
install_rke2 () {


set -o errexit
set -o nounset
#set -o xtrace # Uncomment for verbose output

# Pulls variables for the script to work
source <(cat /usr/lib/rke2/scripts/source/variables)

# Creates and configures Rancher yaml
mkdir -p /etc/rancher/rke2/

cat > /etc/rancher/rke2/config.yaml <<EOF
tls-san:
  - ${server1_hostname}
  - ${server1_ip}
  - ${vip_hostname}.${domain}
  - ${vip_ip}
EOF
echo "Created Config file"

chmod +x /usr/lib/rke2/scripts/install/install.sh
echo "Permissions set to executeable"

/usr/lib/rke2/scripts/install/install.sh
echo "Installed install.sh"

systemctl enable rke2-server.service --now
echo "Enabled the rke2-server.service"

app_name="rke2-canal" namespace="kube-system" container_type="daemonset" check_container_status
app_name="rke2-ingress-nginx-controller" namespace="kube-system" container_type="daemonset" check_container_status
app_name="rke2-coredns-rke2-coredns" namespace="kube-system" container_type="deployment" check_container_status
app_name="rke2-coredns-rke2-coredns-autoscaler" namespace="kube-system" container_type="deployment" check_container_status
app_name="rke2-coredns-rke2-metrics-server" namespace="kube-system" container_type="deployment" check_container_status
}
```
# Your check container_status.sh will look like this
```
 check_container_status() {
     while ! kubectl -n ${namespace} rollout status ${container_type}.apps/${app_name}; do
         sleep 2 && echo "Waiting for ${app_name} to come up"
     done
 }
```
# main.sh should look like this
```
#!/bin/env bash


set -o errexit
set -o nounset
# set -o xtrace # Uncomment for verbose output

# Redirects all standard output to the file /usr/lib/rke2/scripts/install.log
exec > /usr/lib/rke2/scripts/install.log 2>&1 

# sets the evironment variables source to all files in the source directory
source <(cat /usr/lib/rke2/scripts/source/*)

# Calls the function to prepare the server for the installation of RKE2
#prep_server
install_rke2
```
# variables will look something like this
```
# Variables for the cluster configuration
server1_ip=172.16.3.51
server1_hostname=stud3-s1
domain=stud3.soroc.mil
vip_ip=172.16.3.50
vip_hostname=vip

# Environment variables for kubernetes
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
export PATH=$PATH:/var/lib/rancher/rke2/bin:/usr/local/bin
export CRI_CONFIG_FILE=/var/lib/rancher/rke2/agent/etc/crictl.yaml
alias ku=kubectl
alias kuebctl=kubectl
alias k=kubectl
```
# Rke2-install.sh will look like this
```
#!/bin/env bash

set -o errexit

set -o nounset

#set -o xtrace # uncomment for verbose output

chmod +x /usr/lib/rke2/scripts/install/main.sh

/usr/lib/rke2/scripts/install/main.sh &
```