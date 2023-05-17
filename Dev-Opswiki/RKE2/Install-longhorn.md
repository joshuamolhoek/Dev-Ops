# Installing Longhorn

## Distributed Storage

A distributed storage system is infrastructure that can split data across multiple physical servers, and often across more than one data center. We use kubernetes to take advantage of distributed storage

#

There are two main distributed storage solutions you will see in Edge Compute

Both of these distributed storage systems also support `Dynamic Volume Provisioning`in kubernetes. `Dynamic Volume Provisioning` allows storage volumes to be created on-demand instead of administrators needing to manually create volumes for an application.

### **Rook-Ceph**

- Description
  - Uses the `Rook` operator to deploy and manage Ceph in Kubernetes
  - Block Storage
  - Object Storage
  - Filesystem Storage
- Pros
  - Configuration can be fine-tuned at a very granular level
  - High level of performance
- Cons
  - Setup can be very complex
  - Difficult to use for single node clusters because of how the different components of Ceph work together

### **Longhorn**

- Description
  - Longhorn is developed by Rancher Labs specifically for distributed storage on Kubernetes
  - Block Storage
- Pros
  - Easy to install
  - Intuitive UI makes management easy
  - Integration with Rancher
  - Works well for single-node clusters
- Cons
  - Slower performance than Rook-Ceph
  - Errors are slow to recover
  - Does not support filesystem and object storage natively

## Step 1 - Log in to Rancher

## Step 2 - Install apps with existing Helm Charts

When we added our cluster to Rancher, we can manage the applications we install on our downstream clusters from Rancher.  

Rancher has a list of tested applications which can be installed from the `Apps` pane.

Later we will add our own helm chart repository to install applications but for now, we will use the ones included with Rancher.

1. On the left side of the page, select the `Apps` tab

2. In the `Charts` tab, you will see applications supported by Rancher

3. Find `Longhorn` and select it

4. Select `Install` at the top right of the page

5. At the bottom left of the page, **check the box** for customizing install options and select `Next`.

6. Select the tab `Longhorn Default Settings`

7. Check the box `Customize Default Settings` under `Longhorn Default Settings`

- More options to change will now be displayed

8. Change the default replica count to 1

- We are doing this because we are installing it on a single-node cluster and want to conserver resources.  Keep in mind this will cause us to lose redundnancy in the volumes we create for apps.

9. Select `Edit YAML` at the top of the page

- Notice how this looks similar to the `values.yaml` we created in our own Helm Chart.
- The Rancher Helm Chart uses a `questions.yaml` to display the options in the values yaml in the UI
  - We will add a `questions.yaml` in the next helm chart lab

10. Select `Next`

11. Select `Install`

- A log will appear for the installation of Rancher

12. Once the install is complete, a new tab for `Longhorn` will appear on the left side of the page. Select this tab.

- You may have to clear out namespaces at the top of the page and refresh the page for this to appear.

13. Select the link of the `Longhorn` tab

- This will open a new tab and bring you to the Longhorn management UI

## Step 3 - Complete Lab

### Close Lab issue in Gitlab

- Open your student project in GitLab.

- Find the issue that covers this lab.

- Add comments to the issue indicating that you have finished the lab.

- Close the issue.
```