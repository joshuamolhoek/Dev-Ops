# Add cluster to Rancher

Rancher is a management UI where we can manage all the resources we have been managing from the CLI

#

## Step 1 - Log in to Rancher

In your browser, navigate to `https://<rancher-hostname>`

Log in to Rancher with your student username and password for Rancher

## Step 2 - Add an existing cluster

There will not be any clusters for your user profile currently.

Rancher can either be used to add an existing cluster or build a cluster for you on a node which does not have kubernetes installed already.  We will Import an existing cluster.

1. Click the `Import Existing` blue button above the `Clusters` box

2. Select the `Generic` option

3. Name the cluster `stud<student-number>`

4. At the bottom right of the screen, click the `Create` button

5. Copy the **SECOND** option which begins with `curl --insecure`

6. Log in to Server 1

7. Run the command you copied from the the Rancher UI as `root`

8. A new pod should be creating in the cattle-system namespace. Once this completes, select the 3 lines at the top left of the Rancher UI and navigate to the home page

9. The cluster you just added can now be seen under `Clusters`. Select your cluster

## Step 3 - Treasure Hunt

Take this time to get comfortable with the Rancher UI and compare to what you are used to seeing from the commandline using kubectl

Use the tabs on the left side of the page to find the following resources

- View `pods` in the kube-system namespace
  - View the logs for the `rke2-canal` `pod`
- View `deployments` in the kube-system namespace
  - View the `deployment` `rke2-coredns-rke2-cordns` in `YAML` format and compare to what you saw when we created our own deployment manifest file
- View the `services` in the kube-system namespace
  - View the `service` `rke2-coredns-rke2-cordns` in `YAML` format and compare to what you saw when we created our own deployment manifest file

## Step 4 - Complete Lab

### Close Lab issue in Gitlab

- Open your student project in GitLab.

- Find the issue that covers this lab.

- Add comments to the issue indicating that you have finished the lab.

- Close the issue.
```