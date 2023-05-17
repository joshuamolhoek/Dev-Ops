# Creating a Helm Chart

Helm lets you create templated resources for Kubernetes.  This means you can expose information in your manifest files which can change between clusters, as variables.

## Step 1 - Stay on your student laptop

Open a terminal on your dev laptop

## Step 2 - Build a helm chart

A helm chart is a collection of resources such as services, deployments, ingresses, etc., which at a minimum needs a `Chart.yaml` and a `templates` directory. We will also be using a `values.yaml` file to specify values in our resources.

Create a `helm-charts` directory:

```shell
mkdir ~/helm-charts
```

Create a directory for the apache application we are creating a helm chart for

```shell
cd helm-charts

mkdir stud<student-number>-apache
```

Create the templates directory and Chart.yaml

```shell
cd stud<student-number>-apache/

mkdir templates
touch Chart.yaml
touch values.yaml
```

The helm-charts directory should look like this:

```shell
helm-charts/
      studX-apache/
          templates/
          Chart.yaml
          values.yaml
```

Add the following to the `Chart.yaml`

```shell
name: stud<student-number>-apache
version: 1.0.0
```

The `templates/` directory will contain all the resources needed for deploying our application.

Create a resource for your namespace, deployment, and service.

```shell
cd templates

touch apache-deployment.yaml
touch apache-service.yaml
```

The file structure should now look like this:

```shell
helm-charts/
      studX-apache/
          templates/
              apache-deployment.yaml
              apache-service.yaml
          Chart.yaml
          values.yaml
```

Fill in the apache-deployment.yaml and apache-service.yaml with the same contents from the application we deployed in templating Lab_01:

>**NOTE** We will not be using a namespace yaml because we will be creating it using other methods when we deploy our Helm Chart

apache-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apache-stud<student-number>
  namespace: apache-stud<student-number>
spec:
  replicas: 1
  selector:
    matchLabels:
      name: apache-stud<student-number>
  template:
    metadata:
      labels:
        name: apache-stud<student-number>
    spec:
      containers:
      - name: apache-stud<student-number>
        image: https://harbor.vip.ark1.soroc.mil/<stud-number>/apache-server:<tag-with-stud-number>
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
```

apache-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: apache-stud<student-number>-lb
  namespace: apache-stud<student-number>
spec:
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    name: apache-stud<student-number>
  type: LoadBalancer
```

In the templating Lab_01 we created a deployment which will work in a specific namespace with a specific image.  If we wanted to change values for any of the resources, we would have to change it in every resource manifest file.  Helm will do this for us based on what we specify in our `values.yaml`file as long as we add the variables to our helm chart templates.

Add the following to the `values.yaml` file

```yaml
metadata:
  name: apache-stud<student-number>
  namespace: apache-stud<student-number
spec:
  containers:
    image: https://harbor.vip.ark1.soroc.mil/<stud-number>/apache-server:<tag-with-stud-number>
```

Variables which are provided in the `values.yaml` are accessible from the `.Values` object in the template.  

For example if we wanted to use the name of our application specified in our `values.yaml` we would write it as follows

```yaml
{{ .Values.metadata.name }}
```

## Step 3 - Build a deployment template

Open the apache-deployment.yaml and change the name, namespace, and container image to refer to the `values.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.metadata.name }}
  namespace: {{ .Values.metadata.namespace }}
spec:
  replicas: 1
  selector:
    matchLabels:
      name: {{ .Values.metadata.name }}
  template:
    metadata:
      labels:
        name: {{ .Values.metadata.name }}
    spec:
      containers:
      - name: {{ .Values.metadata.name }}
        image: {{ .Values.spec.containers.image }}
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
```

## Step 4 - Create the service template

Repeat the same process from step 3 for the apache-service.yaml

- Create variables like we did in Step 3 for the following information:
  - `name:`
  - `namespace:`

## Step 5 - Install using helm

Ensure your kube-config file is correct and allows you to access your clusters

```shell
kubectl get nodes
```

Delete the resources you created in templating Lab 01 if they still exist

```shell
kubectl delete deployment -n apache-stud<student-number> apache-stud<student-number>
kubectl delete service -n apache-stud<student-number> apache-stud<student-number>-lb
kubectl delete namespace apache-stud<student-number>
```

Move to the base directory of your helm chart

```shell
cd ~/helm-charts/stud<student-number>-apache/
```

Create the Namespace with kubectl:

```shell
kubectl create ns apache-stud<student-number>
```

Run install the helm chart

**THIS WILL INSTALL THE HELM CHART ON THE CLUSTER YOUR KUBECONFIG IS CONFIGURED FOR**

```shell
helm install apache-stud4 .
```

Check the kubernetes cluster to make sure your app has been deployed

Check the namespace:

```shell
kubectl get namespace
```

Check the deployment:

```shell
kubectl get pods -n apache-stud<student-number>
```

Check the service and get the IP address assigned by kube-vip:

```shell
kubectl get svc -n apache-stud<student-number>
```

You can also view the helm project with helm

```shelll
helm ls
```

## Step 6 - Uninstall using helm

The helm chart can be unistalled with

```shell
helm uninstall <release name shown in the previous command>
```

Then delete the Namespace:
**We still need to delete the namespace using kubectl since we created it using kubectl**

```shell
kubectl delete ns apache-stud<student-number>
```
```