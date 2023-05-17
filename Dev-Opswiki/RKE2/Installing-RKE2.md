# Installing RKE2
> the following steps shows you how to install RKE2 onto your server node
# Installing RKE2 on the first server node

Using the script from Lab 03, the server node has already been prepared for the installation of RKE2.

#

## Step 1 - Log in to the Server Node

The following steps will be completed on the Server node VM

All of the configurations we will make to install RKE2 will be completed as the root user.

To change to the root user run the following command:

```shell
sudo su
```

#

## Step 2 - Add environment variables

We already added the environment variables for kubernetes to the variables file in the last lab. Now we can source that directory using the root users `.bashrc` file.

Edit the `.bashrc` file

```shell
vim /root/.bashrc
```

Add the following line to the end of the file:

```shell
source <(cat /usr/lib/rke2/scripts/source/*)
```

## Step 3 - Create config.yaml

Create the directories you will need to install RKE2

```shell
mkdir -p /etc/rancher/rke2/
```

The config.yaml is used by RKE2 to configure the cluster when it is started without needing to run commandline arguments.  For additional options and configuration methods see <https://docs.rke2.io/install/install_options/install_options/>  

```shell
vim /etc/rancher/rke2/config.yaml
```

Input the correct information for the cluster you are building:

```yaml
tls-san:
  - <Server1 hostname>
  - <Server1 IP>
  - <VIP hostname>.<domain>
  - <VIP IP>
```

>**NOTE:** Kube-VIP will be installed later.  Adding it the tls-san option in the config.yaml allows Kube-VIP to load balance traffic coming to the server when there are multiple server nodes configured with High Availability (HA).

## Step 4 - Edit the `install.sh` file

The `install.sh` file we downloaded from Rancher in RKE2 - Lab 01 will need to be edited for our specific installation

#

Open the script for editing

```shell
vim /usr/lib/rke2/scripts/install/install.sh
```

There are 2 variables we will set at the beginning of the script: `INSTALL_RKE2_METHOD="tar"` and `INSTALL_RKE2_ARTIFACT_PATH=/usr/lib/rke2/scripts/install/`

Add the two variables to script and save the file.

```shell
#!/bin/sh

set -e

if [ "${DEBUG}" = 1 ]; then
    set -x
fi

INSTALL_RKE2_METHOD="tar"

INSTALL_RKE2_ARTIFACT_PATH=/usr/lib/rke2/scripts/install/

# Usage:
#   curl ... | ENV_VAR=... sh -
#       or
#   ENV_VAR=... ./install.sh
#

--- snippet ---
```

## Step 5 - Install RKE2

The **install.sh** script packaged with the Edge Compute rke2 airgap bundle will move all the files you just unpacked to the correct directories in order to start the RKE2 service

Make the shell script executable:

```shell
chmod +x /usr/lib/rke2/scripts/install/install.sh
```

Run the script:

```shell
/usr/lib/rke2/scripts/install/install.sh
```

### Enable and start the RKE2 Server service

> **NOTE** without the "--now" option, you would also need to run "systemctl start rke2-server.service" to start it

To start the rke2-service:

```shell
systemctl enable rke2-server.service --now
```

### Monitor the RKE2 installation

Open a new terminal

### Watch the journal logs

Watch the journal logs for the rke2-server service and wait for the installation to complete.

```shell
journalctl -u rke2-server -f
```

The errors will stop once the installation is complete.

## Step 6 - Verify the installation has completed

It may take a 3-5 minutes for the installation to complete depending on the resources the server has.

Check to make sure your node is ready

```bash
kubectl get nodes
```

## Uninstalling RKE2

In the development environment, starting with a clean slate when testing the things we do is extremely important.  In this course we have the option to revert our virtual machines to a snapshot.  That will not always be the case.  It is very likely you will see Kubernetes deploy on **bare-metal** rather than Virtual Machines in order to take advantage of the resources available on the hardware.

#

When RKE2 is installed, it places a few shell scripts in /usr/local/bin/.  `/usr/local/bin/` is also added to the path of the operating system so we do not need to navigate here to run it. The two scripts we use here are named `rke2-killall.sh` and `rke2-uninstall.sh`

To uninstall rke2, it is usually best to make sure everything is stopped

```shell
rke2-killall.sh
```

The run the uninstall script

```shell
rke2-uninstall.sh
```

This script might not get rid of everything though depending on if we have modified directories used by rke2 so it is always good to run the following commands as well:

```shell
rm -rf /var/lib/rancher/
rm -rf /run/k3s/
```

Additionally if Longhorn or Rook-Ceph was provisioned on the node, you will also need to remove the directories used by these apps.
> **NOTE** If you attached drives to be used by storage, you will also need to provision those drives before they can be used again.

```shell
rm -rf /var/lib/longhorn/
rm -rf /var/lib/rook/
```

Finally, remove the `source` line we added to the `/root/.bashrc` file

You may now test and run your script again.



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

# main.sh should look like this

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

Once you have everything run your script using the following command

- /usr/lib/rke2/scripts/rke2-install.sh


# Troubleshooting
> All notes for troubleshooting your cluster
# Everything to uninstall when resetting your node

- /usr/local/bin/rke2-killall.sh
- /usr/local/bin/rke2-uninstall.sh
- rm -rf /var/lib/rancher
- rm-rf /run/k3s/
- /usr/local/bin/rke2-uninstall.log
