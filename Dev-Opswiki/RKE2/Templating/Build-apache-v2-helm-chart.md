# Creating Apache V2 Helm Chart

## Step 1 - Stay on your student laptop

On your student laptop, navigate to the helm-charts directory we used for our helm chart.

```shell
cd ~/helm-charts/stud<student-number>/
```

Create a new directory to store the Version 1 chart we made

```shell
mkdir 1.0.0
```

Copy all files in the directory into this new directory and clean up the templates directory

```shell
cp -r templates/ 1.0.0/
mv Chart.yaml values.yaml 1.0.0/
rm -r templates/
```

The directory structure will now look like this

```shell
helm-charts/
      studX-apache/
          1.0.0/
              templates/
                  apache-deployment.yaml
                  apache-namespace.yaml
                  apache-service.yaml
              Chart.yaml
              values.yaml
```

Create a new directory for version 2

```shell
mkdir 2.0.0
```

Copy all files from Version 1 into the new version 2 directory so we have a baseline

```shell
cp -r 1.0.0/* 2.0.0/
```

The directory structure will now look like this

```shell
helm-charts/
      studX-apache/
          1.0.0/
              templates/
                  apache-deployment.yaml
                  apache-namespace.yaml
                  apache-service.yaml
              Chart.yaml
              values.yaml
          2.0.0/
              templates/
                  apache-deployment.yaml
                  apache-namespace.yaml
                  apache-service.yaml
              Chart.yaml
              values.yaml
```

All changes to our application will now be done on the files within the `2.0.0/` directory

## Step 2 - Change the Chart.yaml

Change the version number in the Chart.yaml to `2.0.0`

## Step 3 - Create a questions.yaml

in the same directory as your `values.yaml` and `Chart.yaml`, create a new file named `questions.yaml`

The `questions.yaml` is used to display configuration in the Rancher UI instead of needing to edit the values.yaml.

```shell
<Repository-Base>/
 │
 ├── charts/
 │   ├── <Application Name>/	  # This directory name will be surfaced in the Rancher UI as the chart name
 │   │   ├── <App Version>/	  # Each directory at this level provides different app versions that will be selectable within the chart in the Rancher UI
 │   │   │   ├── Chart.yaml	  # Required Helm chart information file.
 │   │   │   ├── questions.yaml	  # Form questions displayed within the Rancher UI. Questions display in Configuration Options.*
 │   │   │   ├── README.md         # Optional: Helm Readme file displayed within Rancher UI. This text displays in Detailed Descriptions.
 │   │   │   ├── requirements.yml  # Optional: YAML file listing dependencies for the chart.
 │   │   │   ├── values.yml        # Default configuration values for the chart.
 │   │   │   ├── templates/        # Directory containing templates that, when combined with values.yml, generates Kubernetes YAML.
```

```yaml
categories:
- apache-stud<student-number>
questions:
- variable: metadata.name
  default: apache-stud<student-number>
  type: string
  group: "Apache Settings"
  description: "Name of the application"
- variable: metadata.namespace
  default: apache-stud<student-number>
  type: string
  group: "Apache Settings"
  description: "Namespace the resources will deploy into"
- variable: spec.containers.image
  default: orko/apache-server:<tag-with-stud-number>
  type: string
  group: "Apache Settings"
  description: "image name and version for the apache server"
```

The directory will look like this when you are complete

```shell
helm-charts/
      studX-apache/
          1.0.0/
              templates/
                  apache-deployment.yaml
                  apache-namespace.yaml
                  apache-service.yaml
              Chart.yaml
              values.yaml
          2.0.0/
              templates/
                  apache-deployment.yaml
                  apache-namespace.yaml
                  apache-service.yaml
              Chart.yaml
              values.yaml
              questions.yaml
```

## Step 4 - Package the Helm Chart

navigate to the helm chart directory we use in the previous lab:

```shell
cd ~/helm-charts
```

list the files in the directory.

You should see the apache-stud<student-number> directory

```shell
ll
```

Using helm, package the helm chart

```shell
helm package apache-stud<student-number>/
```

List the files in your directory again

You should now see a new tgz file with the name and version of your app

```shell
ll
```

Upload this new `tgz` into Harbor

```