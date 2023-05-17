
# Installing Kube-vip

One of the applications used for JCU Kubernetes clusters to improve the functionality of the platform, is Kube-vip. kube-vip provides Kubernetes clusters with a virtual IP and load balancer for both the control plane (for building a highly-available cluster) and Kubernetes Services of type LoadBalancer without relying on any external hardware or software.

More information on Kube-vip can be found on their website [here](https://kube-vip.io/)

#

## Step 1 - Log into Server 1

#

## Step 2 - Verify cluster

#

Ensure Lab 06 is completed and RKE2 is functioning on your server nodes.

## Step 3 - Apply the manifest

#

A manifest is a kubernetes resource in jaml or json format with a description of all the components of the resources being deployed.

RKE2 provides additional functionality allowing us to put files into the directory `/var/lib/rancher/rke2/server/manifests/` and RKE2 will automatically deploy the resources we put there.

### Create the Kube-vip RBAC file

#

Create the `kube-vip-rbac.yaml` and open for editing (This file will create the resources for the kube-vip ServiceAccount, ClusterRole, and ClusterRoleBinding):

```bash
vim /var/lib/rancher/rke2/server/manifests/kube-vip-rbac.yaml
```

Paste the following into the kube-vip-rbac.yaml file:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-vip
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  name: system:kube-vip-role
rules:
  - apiGroups: [""]
    resources: ["services", "services/status", "nodes"]
    verbs: ["list","get","watch", "update"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["list", "get", "watch", "update", "create"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: system:kube-vip-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-vip-role
subjects:
- kind: ServiceAccount
  name: kube-vip
  namespace: kube-system
```

### Create the daemonset

Next we will create the daemonset

```bash
vim /var/lib/rancher/rke2/server/manifests/kube-vip.yaml
```

Add the following to this file **BE SURE TO CHANGE VALUES SPECIFIC TO YOUR CLUSTER**

The name of the network interface you are using can be found using the nmtui, ip, or nmcli commands.  For example: `nmcli device status`

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  creationTimestamp: null
  name: kube-vip-ds
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: kube-vip-ds
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: kube-vip-ds
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                - <s1_hostname>
      containers:
      - args:
        - manager
        env:
        - name: vip_arp
          value: "false"
        - name: vip_interface
          value: lo
        - name: port
          value: "6443"
        - name: vip_cidr
          value: "32"
        - name: cp_enable
          value: "true"
        - name: cp_namespace
          value: kube-system
        - name: vip_ddns
          value: "false"
        - name: svc_enable
          value: "true"
        - name: vip_startleader
          value: "false"
        - name: vip_addpeerstolb
          value: "true"
        - name: vip_localpeer
          value: <s1_hostname>:<s1_ip>:10000
        - name: bgp_enable
          value: "true"
        - name: bgp_routerid
        - name: bgp_routerinterface
          value: "<network_interface>"
        - name: bgp_as
          value: "65000"
        - name: bgp_peeraddress
        - name: bgp_peerpass
        - name: bgp_peeras
          value: "65000"
        - name: bgp_peers
          value: <s1_ip>:65000::false
        - name: address
          value: <vip_ip>
        image: plndr/kube-vip:v0.3.5
        imagePullPolicy: Always
        name: kube-vip
        resources: {}
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
            - NET_RAW
            - SYS_TIME
      hostNetwork: true
      nodeSelector:
        node-role.kubernetes.io/master: "true"
      serviceAccountName: kube-vip
      tolerations:
      - effect: NoSchedule
        key: node-role.kubernetes.io/master
        operator: Exists
```

### Create the cloud controller

The cloud controller will act as a DHCP server for services with the type LoadBalancer

#

Next create the kube-vip cloud controller and all of it's resources:

```bash
vim /var/lib/rancher/rke2/server/manifests/kube-vip-cloud-controller.yaml
```

Add the following:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-vip-cloud-controller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  name: system:kube-vip-cloud-controller-role
rules:
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["get", "create", "update", "list", "put"]
  - apiGroups: [""]
    resources: ["configmaps", "endpoints","events","services/status", "leases"]
    verbs: ["*"]
  - apiGroups: [""]
    resources: ["nodes", "services"]
    verbs: ["list","get","watch","update"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: system:kube-vip-cloud-controller-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-vip-cloud-controller-role
subjects:
- kind: ServiceAccount
  name: kube-vip-cloud-controller
  namespace: kube-system
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: kube-vip-cloud-provider
  namespace: kube-system
spec:
  serviceName: kube-vip-cloud-provider
  podManagementPolicy: OrderedReady
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: kube-vip
      component: kube-vip-cloud-provider
  template:
    metadata:
      labels:
        app: kube-vip
        component: kube-vip-cloud-provider
    spec:
      containers:
      - command:
        - /kube-vip-cloud-provider
        - --leader-elect-resource-name=kube-vip-cloud-controller
        image: kubevip/kube-vip-cloud-provider:0.1
        name: kube-vip-cloud-provider
        imagePullPolicy: Always
        resources: {}
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      serviceAccountName: kube-vip-cloud-controller
```

The last resource we will create is a config map to give kube-vip a range of IP addresses to work with:

```bash
vim /var/lib/rancher/rke2/server/manifests/kube-vip-configmap.yaml
```

Add the following **CHANGE VALUES FOR YOUR CLUSTER**

```yaml
apiVersion: v1
data:
  range-global: <vip_ip_range>
kind: ConfigMap
metadata:
  annotations:
    provider: kubevip
  name: kubevip
  namespace: kube-system
```

## Step 4 - Verify installation

The kube-vip daemonset and cloud provider statefulset will be deployed in the kube-system namespace.  Verify both pods are deployed.

```bash
kubectl get pods -n kube-system
```

## Step 5 - Update Gitlab

1. Commit your changes to Gitlab

2. Add steps for installing kube-vip to your wiki

- If the guide was missing any steps add them to your documentation
- If you ran into any issues during this lab, place a **description** of the issue **and** the **solution** to the problem in the Troubleshooting section of the wiki

## Step 6 - Complete Lab

### Close Lab issue in Gitlab

- Open your student project in GitLab.

- Find the issue that covers this lab.

- Add comments to the issue indicating that you have finished the lab.

- Close the issue.
```