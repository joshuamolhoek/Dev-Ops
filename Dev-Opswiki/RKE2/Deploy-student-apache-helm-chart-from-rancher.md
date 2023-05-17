# Deploy Student Apache Helm Chart from Rancher

## Step 1 - Log in to Rancher

## Step 2 - Install application

Follow the same steps you used to install Longhorn, but for your student apache helm-chart

**On the page for selecting a namespace, you may enter a namespace to install the application into and Rancher will create it.** Use the same namespace we used in previous labs for your apache application

# make sure to run the following command on your server node to create the namespace
```
kubectl create ns <name that you choose>
```

## Step 3 - Delete Application

Uninstalling an application which was installed through Rancher Apps is simple.

1. Under the `Apps` tab, select `Installed Apps`

2. Check the box to the left of your apache application

3. At the top, select `Delete`

# Once you have the pod up and running on your cluster type the following command to be able to see what external ip you will use to connect
```
kubectl get service -A
however you can use the following to access it internal to the cluster
172.16.3.51:<port of the Load balancer>
```