# Uploading a helm chart to Harbor

Previously we have installed helm using the binary on our laptops to install our apache app, created a manifest to use rke2 to install the apache app, and then installed the Longhorn Helm Chart using Rancher's repositories.  In this lab, we will upload our Apache application's Helm Chart to our Harbor repository so we can install it using Rancher.

---

## Step 1 - Log in to Harbor

## Step 2 - Upload Helm Chart

1. Select your student Project

1. Select The `Helm Charts` tab in the project

1. Select `Upload`

1. Navigate to the directory on your laptop where we packaged the Helm Chart into a `.tgz` file and select that `.tgz` file

1. Your chart with the name of your helm-chart will now be displayed in Harbor.

From here you can now view all the information from the chart including the version and the `values.yaml` specified in the chart when it was created.

```