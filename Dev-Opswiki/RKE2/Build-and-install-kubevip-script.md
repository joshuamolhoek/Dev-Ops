# Automate the private repo and kube-vip install

## Step 1 - Add private repo to RKE2 installation

In lab 06, the cluster was connected to an external repository after it was already started.  The `registries.yaml` can also be placed in the `/etc/rancher/rke2/` directory prior to the initial startup of the cluster.

#

Add the private repo to the `install_rke2` function

- The script must create the `registries.yaml` before starting the rke2-server service
  - Every action written into the script will have a comment explanation before the action
  - Every action written into the script will print the succesfull completion of the action using `echo`
  - the `registries.yaml` will have variables for the `Harbor hostname` and `domain`

## Step 2 - Install Kube-vip

Create a new function named `install_kube_vip` in a new file named `~/rke2/scripts/source/install_kube_vip.sh`

- Every action written into the script will have a comment explanation before the action
- Every action written into the script will print the succesfull completion of the action using `echo`
- Create all files nessecary for the installation of kube-vip
  - Create variables for all pieces of information you needed to change on the daemonset.

## Step 3 - Update Gitlab

1. Commit your changes to Gitlab

2. Add steps for how to package and run your script to the wiki

- If the guide was missing any steps add them to your documentation
- If you ran into any issues during this lab, place a **description** of the issue **and** the **solution** to the problem in the Troubleshooting section of the wiki

## Step 4 - Complete Lab

### Close Lab issue in Gitlab

- Open your student project in GitLab.

- Find the issue that covers this lab.

- Add comments to the issue indicating that you have finished the lab.

- Close the issue.
```