# Manual install
> All of these steps will be done on your student laptop.
assuming you already have your cd ~/stud3-kubernetes directory made
then use the following command in that directory
mkdir -p rke2/scripts/install/

## Step 1 - Build VMs
# RKE2 Preparation

Kubernetes can be deployed in many different configurations.  Most of the time JCU will be deploying the RKE2 on a bare-metal system instead of virtual machines.  This is done to conserve resources and reduce the O&M cost of managing a hypervisor as well as an Operating System.

# Planning a configuration

RKE2 calls the master node `servers` and the worker nodes `agents`

There are a few configurations you might see when using RKE2.  These following examples are a few of the more common configurations but there are no hard guidlines.  It is important to consider the hardware and resources available when planning your cluster.

## Single Node

Usually used on small devices or when multiple nodes are not available

#

- 1 Server node

### Pros

- Lightweight
- Easy to manage

### Cons

- Pods are not highly-available (HA)
- replication of pods is limited to resources on the one node
- Unless the device has a lot of resources, applications are limited
- Workloads cannot be separated to run on agent nodes because there are none

## 2 Nodes

Usually used conceptually for training or for testing when server and agent functionality require testing but can also be used if only two devices are available

#

- 1 Server node
- 1 Agent node

> **NOTE** RKE2 should not be deploy with two server nodes because Kubernetes does not deploy countermeasures against `split-brain`.
>
> - **Split brain** is a term used to describe a situation when the master componenents on a server node mistakely believe themselves to be the leader at the same time.

### Pros

- Pods running on the cluster are highly avaiable

### Cons

- master components are not highly available
- If the server node is lost, the agent node will lose access to updates from the master components.

## 3 Nodes

This configuration is usually used for a cluster which has the primary purpose of managing other nodes using Rancher but may also be used when hardware is limited to 3 nodes

#

- 3 Server nodes

### Pros

- Master components and workload pods are highly available

### Cons

- Workloads cannot be separated to run on agent nodes because there are none

## 4+ Nodes

This configuration is used on builds with more than 3 nodes available to install on. This could be anything from a 4 Node HPE EL8000 to multiple clustered datacenter servers

#

- 3 Server Nodes
- 1+ Agent Node

### Pros

- Master and workload components are highly available
- Replication can take place on many more nodes

### Cons

- Higher management cost

## Additional considerations

- Workloads can be told to run only on Agent nodes but depending on hardware, may need to run on Server nodes as well to take advantage of hardware available
- Take the hardware into account if you have different types when planning the configuration
  - Low CPU and Memory: Use these devices for server nodes
  - GPU: use this device as an agent node if possible
  - Large amount of storage: Plan to use these nodes for distributed storage for the rest of your cluster

## Step 1 - Build VMs

Because we are in a learning environment, we will be using VMs for deploying RKE2 using the following configuration

- 1 Server Node
- 1 Agent Node

Using `Rocky 8.4` build 2 VMs using the guide from the `Mod 1 Linux Lab 01`

- Below is all the additional information you will need to build the VMs and install RKE2 throughout this Module


"STUDENT 3 SHOWN BELOW"

"X" indicates first, second, etc. IP address assigned from Range

# Server 1
- CPU: 4
- RAM: 8
- Disk: 100 GB
- IP: X.X.X.51
- Hostname: stud<student-number>-s1

# Agent 1
- CPU: 4
- RAM: 8
- Disk: 100 GB
- IP: X.X.X.52
- Hostname: stud<student-number>-a1

# Cluster Variables
- Domain: `stud<student-number>.soroc.mil`
- VIP IP: X.X.X.50
- VIP Hostname: vip
- VIP Range: X.X.X.54-X.X.X.59
- coredns IP: X.X.X.53
- Upstream DNS: 10.10.44.50 (Use this one for DNS when building VMs)
- Harbor Hostname: harbor.vip.ark1.soroc.mil
- Rancher Hostname: rancher.vip.ark1.soroc.mil

## Now that your server and agent are set up follow the steps below to start the rke2 install process

# Preparing Air-gapped installation files for RKE2
- log in to your student laptop(all prep will be done here)
- Make the ~/stud3-kubernetes/rke2/ directory by using the following command
- mkdir -p ~/stud3-kubernetes/rke2/
# Prepare the standard file structure
- it will look like the example below
```shell
└── rke2
    └── scripts
        └── install
            ├── install.sh
            ├── rke2-images.linux-amd64.tar.zst
            ├── rke2.linux-amd64.tar.gz
            └── sha256sum-amd64.txt
```
- now make the directory below
```
mkdir -p rke2/scripts/install/
```
- go to the directory we just created
```
cd ~/stud3-kubernetes/rke2/scripts/install/
```
- use the following commands to download the files
```
curl -OLs https://github.com/rancher/rke2/releases/download/v1.22.9%2Brke2r2/rke2-images.linux-amd64.tar.zst

curl -OLs https://github.com/rancher/rke2/releases/download/v1.22.9%2Brke2r2/rke2.linux-amd64.tar.gz

curl -OLs https://github.com/rancher/rke2/releases/download/v1.22.9%2Brke2r2/sha256sum-amd64.txt

curl -sfL https://get.rke2.io --output install.sh | INSTALL_RKE2_VERSION=v1.22.9+rke2r2

```
- then make sure you have 4 files in the ~/rke2/scripts/install/ directory
```
[stud3 install]$ ll
total 832932
-rw-rw-r--. 1 dev dev     21047 May 30 11:30 install.sh
-rw-rw-r--. 1 dev dev 762982325 May 30 11:31 rke2-images.linux-amd64.tar.zst
-rw-rw-r--. 1 dev dev  89907802 May 30 11:28 rke2.linux-amd64.tar.gz
-rw-rw-r--. 1 dev dev      3626 May 30 11:28 sha256sum-amd64.txt

```
- log in to your server node
- type the following command to change to the root user
```
sudo su
```
- create a new file by using the following command
```
vim /etc/NetworkManager/conf.d/rke2-canal.conf
```
- in that vim window make sure you add the following
```
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*

```
- then reload the network manager using the following command
```
systemctl reload NetworkManager

```
- Now we will configure the /etc/hosts file
- open the /etc/hosts file
```
vim /etc/hosts

```
- using the template below use YOUR STUDENT INFORMATION for your cluster variables
```
127.0.0.1   localhost
<Server1 IP> <Server1 hostname> <Server1 hostname>.<domain>
<VIP IP> <VIP hostname> <VIP hostname>.<domain>

```
- add environment variables for the root user, open the /root/.bashrc file
```
vim /root/.bashrc

```
- add the following to that file
```
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
export PATH=$PATH:/var/lib/rancher/rke2/bin:/usr/local/bin
export CRI_CONFIG_FILE=/var/lib/rancher/rke2/agent/etc/crictl.yaml
alias ku=kubectl
alias kuebctl=kubectl
alias k=kubectl
```
- Now use the following command to source your current shell session
```
source /root/.bashrc
```
- Now open the SELinux config file for editing using the following command
```
vim /etc/selinux/config
```
- make the following change to your file
```
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
- use the following command to make sure that it updated
```
setenforce permissive

getenforce
# Should say Permissive

```
- Now we will disable firewalld using the following command
```
systemctl disable firewalld --now
```
# create a directory to hold files you will source as environment variables on your server
```
mkdir -p ~/stud3-kubernetes/rke2/scripts/source
```
# you file structure should look like the one below
```
rke2/
    scripts/
        source/
        install/
            rke2-images.linux-amd64.tar.zst
            rke2.linux-amd64.tar.gz
            sha256sum-amd64.txt
            install.sh
```
# Now add the following initiation script
```
vim ~/<student-project>/rke2/scripts/rke2-install.sh
```
for every script you write make sure this is at the top

```
#Example
#!/bin/env bash

set -o errexit
set -o nounset
#set -o xtrace # Uncomment for verbose output
```
Now we will ensure the main.sh script is set to be executable and run it with the following:
```
#These will be added to rke2-install.sh
chmod +x /usr/lib/rke2/scripts/install/main.sh

/usr/lib/rke2/scripts/install/main.sh &

```
Now your script will look like the following
```
#!/bin/env bash

set -o errexit
set -o nounset
#set -o xtrace # Uncomment for verbose output

chmod +x /usr/lib/rke2/scripts/install/main.sh

/usr/lib/rke2/scripts/install/main.sh &

```
make the rke2-install.sh executable
```
chmod +x ~/rke2/scripts/rke2-install.sh

```

# Build the main.sh script
```
vim ~/<student-project>/rke2/scripts/install/main.sh
```
your script should look like this

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
prep_server

```

The function `prep_server` is the function which will be used to prepare the server for the installation of RKE2

# The following template is the one we will use to create our templates

```
#EXAMPLE
function_name () {
    set -o errexit
    set -o nounset
    # set -o xtrace # Uncomment for verbose output
    <command>
    <command>
}


```
now create a file named prep_server.sh
```
vim ~/<student-project>/rke2/scripts/source/prep_server.sh

```
Now that we have created the function, we need to go back and look at the commands we ran to prepare the server.

The first thing we did in the previous lab was create an interface configuration for the RKE2 cni by adding the following line to a new file named `/etc/NetworkManager/conf.d/rke2-canal.conf` with the contents:
```
#Example
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*

```
There are a few ways to automate this in bash, but we will use redirects to tell bash when to begin and terminate the file.

```
#EXAMPLE
cat >> /file/path/filename.ext <<EOF
<file contents>
EOF

```
To make bash create this file for us in the script, can write it as follows:
```
cat > /etc/NetworkManager/conf.d/rke2-canal.conf <<EOF
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
EOF

```
***NOTE*** Be sure to indent every new line in the function or it will not be read with the function and **ALWAYS LEAVE COMMENTS BEFORE A NEW ACTION**

```
prep_server () {
    set -o errexit
    set -o nounset
    # set -o xtrace # Uncomment for verbose output
# Create the NetworkManager cni configuration
    cat > /etc/NetworkManager/conf.d/rke2-canal.conf <<EOF
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
EOF
}

```
There was one more action we had to take to ensure Network Manager applied the config in the previous lab. We also need to tell the function to reload Network Manager

```
prep_server () {
    set -o errexit
    set -o nounset
    # set -o xtrace # Uncomment for verbose output
# Create the NetworkManager cni configuration
    cat > /etc/NetworkManager/conf.d/rke2-canal.conf <<EOF
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
EOF

# Reload NetworkManager to apply the configuration
    systemctl reload NetworkManager
}

```
The next step requires us to create the `hosts` file in the server. This will contain the IP address and hostname of all servers in the cluster. Right now it is just the one.

write out a script to create the file for you:

```
cat > /etc/hosts <<EOF
127.0.0.1   localhost
${server1_ip} ${server1_hostname} ${server1_hostname}.${domain}
EOF
```
create the variable file

```
vim ~/rke2/scripts/source/variables

```

add the following to your variables file
```
#EXAMPLE
# Variables for the cluster configuration
server1_ip=<server1-ip>
server1_hostname=<server1-hostname>
domain=stud<number>.soroc.mil

```
We will also add the kubernetes environment variables to this file and it should look like this when you're done
```
# Variables for the cluster configuration
server1_ip=<server1-ip>
server1_hostname=<server1-hostname>
domain=soroc.mil

# Environment variables for kubernetes
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
export PATH=$PATH:/var/lib/rancher/rke2/bin:/usr/local/bin
export CRI_CONFIG_FILE=/var/lib/rancher/rke2/agent/etc/crictl.yaml
alias ku=kubectl
alias kuebctl=kubectl
alias k=kubectl

```
Set SELinux and firewalld (For training purposes only)

The final step for the preparation of the server will be to overwrite the SELinux file with our permissive setting, set the running SELinux status to permissive and disable firewalld
```
cat > /etc/selinux/config <<EOF
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
EOF

```
now run the following commands
```
setenforce 0
systemctl disable --now firewalld

```
now you can add it to your prep_server function:
```
prep_server () {
    set -o errexit
    set -o nounset
    # set -o xtrace # Uncomment for verbose output
# Create the NetworkManager cni configuration
    cat > /etc/NetworkManager/conf.d/rke2-canal.conf <<EOF
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
EOF

# Reload NetworkManager to apply the configuration
    systemctl reload NetworkManager

# Overwrite the current hosts file on the server
    cat > /etc/hosts <<EOF
127.0.0.1   localhost
${server1_ip} ${server1_hostname} ${server1_hostname}.${domain}
EOF

# Overwrite the selinux config file
    cat > /etc/selinux/config <<EOF
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
EOF

# Set the running SELinux config
    setenforce 0

# Disable and stop firewalld
    systemctl disable --now firewalld

}

```
For every action the script accomplishes write use the echo command to print a string to the Standard Output and log the completion status.
```
prep_server () {
    set -o errexit
    set -o nounset
    # set -o xtrace # Uncomment for verbose output
# Create the NetworkManager cni configuration
    cat > /etc/NetworkManager/conf.d/rke2-canal.conf <<EOF
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
EOF
    echo "Created NetworkManager cni configuration"

# Reload NetworkManager to apply the configuration
    systemctl reload NetworkManager
    echo "reloaded NetworkManager"

# Overwrite the current hosts file on the server
    cat > /etc/hosts <<EOF
127.0.0.1   localhost
${server1_ip} ${server1_hostname} ${server1_hostname}.${domain}
EOF
    echo "Configured /etc/hosts"

# Overwrite the selinux config file
    cat > /etc/selinux/config <<EOF
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
EOF
    echo "Configured the SELinux config file"

# Set the running SELinux config
    setenforce 0
    echo "Sucessfully set the running selinux config to Permissive"

# Disable and stop firewalld
    systemctl disable --now firewalld
    echo "Successfully stopped and disabled SELinux"

}

```

# Move the installation to the server

navigate to your student project folder
```
cd <student-project>

```
find the filepath to your current directory

```
echo %cd%

```

run a ubuntu container
```
pwd

```

change to command line on student laptop to run

```
docker run -it -v <current-directory>:/mnt ubuntu:latest

```
The -v option creates a bind mount at the specified directory
```
docker run -it -v <host-directory>:<directory-inside-container> <image-name>:<tag>

```
Move to the directory where we mounted our student project
```
cd /mnt

```
Create the compressed archive file:
```
tar -czvf rke2-stud-<student-number>.tar.gz rke2/

```
exit the container
```
exit

```
Return to your gitbash terminal Copy the compressed archive file to your server:
```
scp rke2-stud-<student-number>.tar.gz <username>@<server-ip>:/home/<username>/

```
# ssh into your server node

You should now see the tar gz file in your /home// directory
```
cd /home/<username>/

```

list the directory contents:
```
ls -al

```
The installation of RKE2 reqiures root access so we will change to the root user to run the script.
```
sudo su

```
Unpack the compressed archive file to the `/usr/lib/` directory
```
tar xzvf rke2-stud-<student-number>.tar.gz -C /usr/lib/

```
# Run the script
```
/usr/lib/rke2/scripts/rke2-install.sh

```

watch the status
```
tail -f /usr/lib/rke2/scripts/install.log

```
if it fails run 
```
dos2unix
```
log in to the server and change to the root user
```
sudo su
```
add environment variables

```
vim /root/.bashrc
```
add the following line to the end of the file:

```
source <(cat /usr/lib/rke2/scripts/source/*)
```
create config.yaml
```
mkdir -p /etc/rancher/rke2/
```
The config.yaml is used by RKE2 to configure the cluster when it is started without needing to run commandline arguments. For additional options and configuration methods see https://docs.rke2.io/install/install_options/install_options/
```
vim /etc/rancher/rke2/config.yaml

```
input the correct information for the cluster you are building
```
tls-san:
  - <Server1 hostname>
  - <Server1 IP>
  - <VIP hostname>.<domain>
  - <VIP IP>

```
edit the install.sh file
```
vim /usr/lib/rke2/scripts/install/install.sh

```
add the two variables to scrip and save the file
```
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
install rke2
make the shell script executable
```
chmod +x /usr/lib/rke2/scripts/install/install.sh

```
run the script
```
/usr/lib/rke2/scripts/install/install.sh

```

enable and start the RKE2 Server service

```
systemctl enable rke2-server.service --now

```
use the following command to watch the journal logs
```
journalctl -u rke2-server -f
```
verify the installation has completed

```
kubectl get nodes

```

use the following commands to uninstall rke2

```
rke2-killall.sh
rke2-uninstall.sh
rm -rf /var/lib/rancher/
rm -rf /run/k3s/
rm -rf /var/lib/longhorn/
rm -rf /var/lib/rook/
```
build registries.yaml
```
vim /etc/rancher/rke2/registries.yaml

```
now input the following into that file
```
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
Now restart the rke2-server service
```
systemctl restart rke2-server

```

pull a test image
```
vim busybox-test.yaml

```
**Change the image name and tag** to `busybox:private-repo-test`
We are telling busybox to print a statement with the `echo` command. We will be able to view this in the logs once the pod has deployed.
```
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
apply the manifest file:
```
kubectl apply -f busybox-test.yaml

```
confirm the pod was created
```
kubectl get pods

```

check the logs of the pod to see the output of the command we had it run in the manifest file
```
kubectl logs busybox
```
## Installing Kube-vip

login to server 1

verify that the cluster is up using 
```
kubectl get pods -A
```
create the kube-vip-rbac.yaml
```
vim /var/lib/rancher/rke2/server/manifests/kube-vip-rbac.yaml
```
paste the following into the kube-vip-rbac.yaml file:
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-vip
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  name: system:kube-vip-role
rules:
  - apiGroups: [""]
    resources: ["services", "services/status", "nodes"]
    verbs: ["list","get","watch", "update"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["list", "get", "watch", "update", "create"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: system:kube-vip-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-vip-role
subjects:
- kind: ServiceAccount
  name: kube-vip
  namespace: kube-system\
```
create the daemonset
```
vim /var/lib/rancher/rke2/server/manifests/kube-vip.yaml

```
The name of the network interface you are using can be found using the nmtui, ip, or nmcli commands. For example: `nmcli device status`
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  creationTimestamp: null
  name: kube-vip-ds
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: kube-vip-ds
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: kube-vip-ds
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                - <s1_hostname>
      containers:
      - args:
        - manager
        env:
        - name: vip_arp
          value: "false"
        - name: vip_interface
          value: lo
        - name: port
          value: "6443"
        - name: vip_cidr
          value: "32"
        - name: cp_enable
          value: "true"
        - name: cp_namespace
          value: kube-system
        - name: vip_ddns
          value: "false"
        - name: svc_enable
          value: "true"
        - name: vip_startleader
          value: "false"
        - name: vip_addpeerstolb
          value: "true"
        - name: vip_localpeer
          value: <s1_hostname>:<s1_ip>:10000
        - name: bgp_enable
          value: "true"
        - name: bgp_routerid
        - name: bgp_routerinterface
          value: "<network_interface>"
        - name: bgp_as
          value: "65000"
        - name: bgp_peeraddress
        - name: bgp_peerpass
        - name: bgp_peeras
          value: "65000"
        - name: bgp_peers
          value: <s1_ip>:65000::false
        - name: address
          value: <vip_ip>
        image: plndr/kube-vip:v0.3.5
        imagePullPolicy: Always
        name: kube-vip
        resources: {}
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
            - NET_RAW
            - SYS_TIME
      hostNetwork: true
      nodeSelector:
        node-role.kubernetes.io/master: "true"
      serviceAccountName: kube-vip
      tolerations:
      - effect: NoSchedule
        key: node-role.kubernetes.io/master
        operator: Exists
```
# Create the cloud controller
```
vim /var/lib/rancher/rke2/server/manifests/kube-vip-cloud-controller.yaml
```
add the following to the file
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-vip-cloud-controller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  name: system:kube-vip-cloud-controller-role
rules:
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["get", "create", "update", "list", "put"]
  - apiGroups: [""]
    resources: ["configmaps", "endpoints","events","services/status", "leases"]
    verbs: ["*"]
  - apiGroups: [""]
    resources: ["nodes", "services"]
    verbs: ["list","get","watch","update"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: system:kube-vip-cloud-controller-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-vip-cloud-controller-role
subjects:
- kind: ServiceAccount
  name: kube-vip-cloud-controller
  namespace: kube-system
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: kube-vip-cloud-provider
  namespace: kube-system
spec:
  serviceName: kube-vip-cloud-provider
  podManagementPolicy: OrderedReady
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: kube-vip
      component: kube-vip-cloud-provider
  template:
    metadata:
      labels:
        app: kube-vip
        component: kube-vip-cloud-provider
    spec:
      containers:
      - command:
        - /kube-vip-cloud-provider
        - --leader-elect-resource-name=kube-vip-cloud-controller
        image: kubevip/kube-vip-cloud-provider:0.1
        name: kube-vip-cloud-provider
        imagePullPolicy: Always
        resources: {}
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      serviceAccountName: kube-vip-cloud-controller

```
The last resource we will create is a config map to give kube-vip a range of IP addresses to work with:
```
vim /var/lib/rancher/rke2/server/manifests/kube-vip-configmap.yaml

```
add the following change values for your cluster
```
apiVersion: v1
data:
  range-global: <vip_ip_range>
kind: ConfigMap
metadata:
  annotations:
    provider: kubevip
  name: kubevip
  namespace: kube-system

```
verify installation

```
kubectl get pods -n kube-system
```
## adding the agent node

## Step 1 - Log in to the server node

To add an agent node to the RKE2 cluster, we first need to token generated by the RKE2 server.

The node token is located in the file /var/lib/rancher/rke2/server/node-token

**COPY THE NODE-TOKEN and SAVE IT FOR LATER**

* We will use it for the `config.yaml` on the agent node.

## Step 2 - Move the air gapped dependencies to the agent node

Move all the air gapped dependencies we downloaded for the server to the agent node in `/usr/lib/rke2/scripts/install` and place them in `/usr/lib/rke2/scripts/install` on the Agent node.

## Step 3 - Log in to the Agent node

## Step 4 - Prepare the agent node

* Set SELinux to permissive
* Turn off firewalld
* Check set the NTP server in the /etc/chrony.conf
```
# Server 1 Example
server ${NTP_SERVER_IP} iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
hwtimestamp *
keyfile /etc/chrony.keys
leapsectz right/UTC
logdir /var/log/chrony
```
```
# Agent node Example
server ${NTP_SERVER_IP} iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
hwtimestamp *
keyfile /etc/chrony.keys
leapsectz right/UTC
logdir /var/log/chrony
```
restart chrony service
```
chronyc sources
systemctl status chronyd.service
```
## Create the registries.yaml

Use the same registries.yaml we used for our server node

## Create the config.yaml

Add the following information to your config.yaml

Use the node token you pulled from the server node in **Step 1**
```
server: https://${vip_hostname}.${domain}:9345
token: ${node_token}

```
## Run the RKE2 Agent install script provided by rancher

The `install.sh` file we downloaded from Rancher will also install the Agent service if we tell it to using the following command
```
INSTALL_RKE2_TYPE="agent" /usr/lib/rke2/scripts/install/install.sh
```
run the agent service
```
systemctl enable --now rke2-agent
```
watch the logs
now from the server node you will be able to run
```
kubectl get nodes (will show ready state once its up)
```
