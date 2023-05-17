# Prepare rocky for RKE2
> The following steps prepares your rocky server node
#Preparing Rocky 8.4 for RKE2 installation

## Step 1 - Create a snapshot of the Server Node (s1)

Before you start the steps on this page, create a snapshot of your server node virtual machine

## Step 2 - Log in to the Server Node

The following steps will be completed on the Server node VM

All of the configurations we will make to install RKE2 will be completed as the root user.

To change to the root user run the following command:

```shell
sudo su
```

#

## Step 3 - CNI Configuration

#

We will need to add the configuration settings for NetworkManager to allow the RKE2 cni(Container Network Interface)

#

Create a new file named **rke2-canal.conf** in NetworkManager's configuration directory

```shell
vim /etc/NetworkManager/conf.d/rke2-canal.conf
```

Add the following contents to the file

```shell
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
```

Reload NetworkManager

```shell
systemctl reload NetworkManager
```

## Step 4 - Configure **/etc/hosts** file

#

Open the **/etc/hosts** file in your text editor

```shell
vim /etc/hosts
```

Add the hostnames and IPs for your cluster so they can be reached before DNS is configured

```shell
127.0.0.1   localhost
<Server1 IP> <Server1 hostname> <Server1 hostname>.<domain>
<VIP IP> <VIP hostname> <VIP hostname>.<domain>
```

## Step 5 - Add environment variables for the root user

#

Open the **/root/.bashrc** file with your text editor

```shell
vim /root/.bashrc
```

The following commands need to be added to the end of the **/root/.bashrc** file for management of the RKE2 cluster on the server node.

> **NOTE** You may also add any other aliases for the different ways you commonly mistype "kubectl"

```shell
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
export PATH=$PATH:/var/lib/rancher/rke2/bin:/usr/local/bin
export CRI_CONFIG_FILE=/var/lib/rancher/rke2/agent/etc/crictl.yaml
alias ku=kubectl
alias kuebctl=kubectl
alias k=kubectl
```

Call the **/root/.bashrc** file to be the source for your current shell session

```shell
source /root/.bashrc
```

## Step 6 - Set SELinux to Permissive

#

There are still some issues with RKE2 running on RHEL8 with SELinux enabled so for the purposes of training, we will set this feature to permissive.

#

Open the SELinux config file for editing

```shell
vim /etc/selinux/config
```

Change 'enforcing' to 'permissive'

```shell
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

Change SELinux to 'permissive'
While we have set the SELinux to permissive via the script, it has not yet been set. To ensure that it is set we can do this manually through the following command:

```shell
setenforce permissive

getenforce
# Should say Permissive
```

## Step 7 - Disable firewalld (Add firewalld rules later)

For the purposes of training we will be disabling firewalld

```shell
systemctl disable firewalld --now
```
