# Use only an Agent node for Longhorn storage

Often times when we want to install Kubernetes on large clusters, we want to separate resources so they are not running on the nodes hosting the controlplane components.

#

## Step 00 - Background

Previously, when we installed Longhorn, it was installed using the default settings. There are two problems with that. The default settings used by Longhorn will use the storage space assigned to `/var/lib/longhorn`, which the way our os is configured, will use all space available in `/`.

You can view this with the following commands:

```shell
list all block devices:
lsblk

show logical volumes:
lvs
```

The second issue, is that by default Longhorn will install on all nodes and if we want to only use an Agent for storage, we need to apply labels to the nodes to tell Kubernetes which nodes to use for Longhorn storage.

## Step 1 - Find the defaults from the `values.yaml`

When we installed Longhorn from the Rancher Marketplace, it pulled the values we saw from a Rancher's helm chart repository.  You can find the most recent Lonhorn helm chart here:

[https://github.com/longhorn/charts/tree/master](https://github.com/longhorn/charts/tree/master)

Find the following values in the `values.yaml` file:

```yaml
defaultSettings.createDefaultDiskLabeledNodes
defaultSettings.defaultDataPath
```

Next find the default values and look in the description for helpful notes for these settings by looking at the for them in the `questions.yaml`

Take note of the following information found in the `questions.yaml`

### defaultSettings.createDefaultDiskLabeledNodes

- Default value (boolean):
- Node label to apply (Look at the description):

### defaultSettings.defaultDataPath

- Default value (file path):

## Step 2 - Configure an LVM for Longhorn to use **ON THE AGENT NODE**

Now that we know the default values Longhorn installs with, we can use this information to give us more control over how the application is installed.

#

First we are going to create a logical volume and mount it to the location Longhorn is going to be mounting to on **ONLY** the Agent node.

1. Shut down the Agent node VM

2. Add a new drive of 50 GiB to the Agent node VM

3. Power the Agent node back on and log in to it

4. View available drives.

```shell
lsblk
```

- You should now see a new drive on your server which hasn't been mounted to anything.

5. Create a Persistent Volume on RHEL

```shell  
pvcreate /dev/<drive-to-configure>
```

6. Check PV's  
`pvs`  

7. Create Volume Group  

```shell
vgcreate longhorn-vg /dev/<drive-to-configure>
```

8. Check VG's  
`vgs`  

9. Create Logical Volume and assign it 100% of the drive space

```shell
lvcreate -n longhorn-lv -l 100%FREE longhorn-vg
```

10. Create the ext4 filesystem on the LVM  

```shell
mkfs.ext4 /dev/longhorn-vg/longhorn-lv
```

11. Check Volumes  
`lvs`  

12. Create Longhorn Directory  

```shell
mkdir -p /var/lib/longhorn
```

13. Modify fstab  

```shell
cat >> /etc/fstab << EOF
/dev/mapper/longhorn--vg-longhorn--lv /var/lib/longhorn ext4     defaults 0 0
EOF
```

14. **IMPORTANT** Mount all volumes in the `fstab`

- IF THIS COMMAND SHOWS ERRORS AND YOU REBOOT, YOUR SERVER WILL NOT BOOT
Use the command `mount -a` to mount all files in the fstab.  If you get errors, your formatting is usually incorrect in the file `/etc/fstab`.

15. Verify the lvm is now mounted

```shell
lsblk
```

## Step 3 - Label nodes in RKE2

Use the label we pulled from the description in the helm chart
**Perform this on S1**

```shell
kubectl label node <agent-node> <label-to-apply>=true
kubectl label node <server-node> <label-to-apply>=false
```

## Step 4 - Install Longhorn

Add your cluster to Rancher and install Longhorn.

- This time, make sure you also check the box to `Create Default Disk on Labelled nodes`

## Step 5 - Verify

- Check the labels on the nodes

```shell
kubectl get nodes --show-labels
```

- Make sure the Server has the longhorn label set to false and the agent has the label set to true

- From the Longhorn UI, you should now see 50Gib available on ONLY the Agent node

## Update Gitlab

- Add steps for configuring drives for Longhorn to your Wiki
```