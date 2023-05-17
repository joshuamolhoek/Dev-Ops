# Deploying an application

## Step 1 - Revert Server 1 to the baseline snapshot

## Step 2 - Log in to Server 1

## Step 3 - Install RKE2

Install RKE2 and kube-vip using the script created in previous labs

## Step 4 - Create a namespace resource

In this example, we will create a namespace, deployment, and service for the application we created with Docker at the beginning of Mod 4

Create a manifest file for the namespace:

```bash
vim apache-namespace.yaml
```

Add the following

**Be sure to fill in your own student information**

```bash
apiVersion: v1
kind: Namespace
metadata:
  name: apache-stud<student-number>
```

Apply the manifest file

```bash
kubectl apply -f apache-namespace.yaml
```

Ensure the namespace has been created:

```bash
kubectl get namespace
```

## Step 5 - Create a deployment resource

Create a manifest file for the deployment

```bash
vim apache-deployment.yaml
```

Add the following to the file:

**Be sure to fill in your own student information**

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

Apply the manifest file

```bash
kubectl apply -f apache-deployment.yaml
```

Ensure the deployment has been created:

```bash
kubectl get deployment -n apache-stud<student-number>
```

## Step 6 - Create a service resource

Because we have installed kube-vip on our cluster we can create a service which will use the kube-vip load-balancer to set a virtual IP address for our application.

Create a manifest file for a service using the load-balancer

```bash
vim apache-service.yaml
```

Add the following:

**Be sure to fill in your own student information**

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

Apply the manifest file

```bash
kubectl apply -f apache-service.yaml
```

Ensure the service has been created:

```bash
kubectl get service -n apache-stud<student-number>
```

This should show an IP address which was assigned by Kube-VIP within the range we assigned to kube-vip with a configmap when we installed it.

This IP address is the IP you will use to access the UI for your application in a browser.

## Step 7 - Verify application connectivity

If all resources have been successfully deployed, you will be able to reach the application in your browser at <http://<apache-ip>>

```