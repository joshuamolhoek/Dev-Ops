# Managing Longhorn

## Step 1 - Open the Longhorn Management Dashboard

In the previous lab, you install Longhorn.  Log into Rancher and find the management dashboard for Longhorn

## Step 2 - Navigating the Longhorn UI

### Dashboard

Information and status of the cluster storage can be viewed at a glance from the dashboard

### Node

This tab will show you the information for the node.  

You can also change settings for the individual nodes here

1. Select the drop down under `Operation` for the node in your cluster

2. Take note of the `Path`

- This is the default path for where longhorn will store data on the operating system of the node it is running on.  If you ever need to add storage, you will need to provision a drive and mount it on the node to match the path in this block.

3. Change `Storage Reserved` to **25**
This is an option to prevent overprovisioning of the storage on the host operating system

### Volume

This is where you can view Persistent Volume Claims and their status

### Recurring Job

Recurring Jobs can be created for backups and snapshots of PVCs

### Backup

View backups created from Jobs

### Settings

Any setting which needs to be changed after the installation can be changed here

## Step 3 - Complete Lab

### Close Lab issue in Gitlab

- Open your student project in GitLab.

- Find the issue that covers this lab.

- Add comments to the issue indicating that you have finished the lab.

- Close the issue.
```