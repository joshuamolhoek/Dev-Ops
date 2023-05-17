## Prerequisites
# make your file tree look like the example below
~/helm-charts/
             /<comex>/
                     /templates/
                                deployment.yaml
                                djangoservice.yaml
                                ingress.yaml
                                persistenvolumes.yaml
                                secret.yaml
                                service.yaml
                                statefulset.yaml
                     Chart.yaml
                     values.yaml


## Step 1 Creating all of the yaml files needed for your django application

# making your Persistent volume claim 
```
vim persistenvolume.yaml
```
the outcome of that file will look like the example below

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  namespace: {{ .Values.metadata.namespace }}
  name: django-db-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: {{ .Values.db.storage }}

```

# making your secret resource
```
vim secret.yaml
```
the outcome of that file will look like the example below

```
apiVersion: v1
kind: Secret
metadata:
  namespace: {{ .Values.metadata.namespace}}
  name: django-secret
data:
  POSTGRES_PASSWORD: {{ .Values.secret.pass }}
```

# Now we will make your postgres service resource
```
vim service.yaml
```

the outcome of that file will look like the example below

```
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.metadata.namespace }}
  name: {{ .Values.db.name }}
spec:
  selector:
    app: {{ .Values.db.name }}
  ports:
    - name: http
      protocol: TCP
      port: 5432
      targetPort: 5432
```
# Now make the statefulset resource yaml with the following requirements
```
- [Statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) resource created
  - `metadata.name` = `django-db`
  - `spec.replicas` = `1`
  - `spec.selector.matchLabels.name` = `django-db`
  - `spec.template.metadata.labels.name` = `django-db`
  - `TCP` port configured with `containerPort: 5432`
  - Statefulset is configured to read the secret from the secret containing the POSTGRES_PASSWORD
  - Statefulset is configured to use the 20GiB PVC for storage and attached to the mountPath `/var/lib/postgresql/data` inside the container with a subpath of `studybud`
  - Set an environment variable called `PGDATA` wtih a value of `/var/lib/postgresql/data/studybud`

```
the outcome of that will look something like the example below
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  namespace: {{ .Values.metadata.namespace }}
  name: {{ .Values.db.name }}
spec:
  selector:
    matchLabels:
      app: {{ .Values.db.name }}
  serviceName: {{ .Values.db.name }}
  replicas: {{ .Values.django.replicas }}
  template:
    metadata:
      labels:
        app: {{ .Values.db.name }}
    spec:
      containers:
      - env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: django-secret
              key: POSTGRES_PASSWORD
        name: {{ .Values.db.name }}
        image: {{ .Values.db.image }}
        ports:
        - containerPort: 5432
          name: tcp
        volumeMounts:
        - name: {{ .Values.db.name }}
          mountPath: /var/lib/postgresql/
      volumes:
        - name: {{ .Values.db.name }}
          persistentVolumeClaim:
            claimName: django-db-claim

```
# Now we will make the deployment resource using the following template

```
- [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) resource created
  - `metadata.name` = `django`
  - `spec.replicas` = `1`
  - `spec.selector.matchLabels.name` = `django`
  - `spec.template.metadata.labels.name` = `django`
  - `TCP` port configured with `containerPort: 80`
  - Deployment is configured to read the secret from the secret containing the POSTGRES_PASSWORD
  - Environment variabled configured to set `DB_HOSTNAME` to the value `django-db`
```
it should look like the example below
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.django.name }}
  namespace: {{ .Values.metadata.namespace }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {{ .Values.django.name }}
  template:
    metadata:
      labels:
        app: {{ .Values.django.name }}
    spec:
      containers:
      - env:
        - name: DB_HOSTNAME
          value: {{ .Values.db.name }}
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              key: POSTGRES_PASSWORD
              name: django-secret
        image: {{ .Values.django.image }}
        name: {{ .Values.db.name }}
        ports:
        - containerPort: 80
          name: 80tcp
          protocol: TCP    
```
now make your ingress resource .yaml using the following template
```
- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) resource created
  - ingress is configured to use the service exposing the Django deployment
  - Ingress must use the vip hostname. Example: django.vip.stud1.soroc.mil
```
Your outcome should look like the following example
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: django-ingress
  namespace: {{ .Values.metadata.namespace }}
spec:
  rules:
  - host: {{ .Values.django.host }}
    http:
      paths:
      - backend:
          service:
            name: django-service
            port:
              number: 80
        path: /
        pathType: Prefix
```

now create your django service .yaml resource using the following template below

```
- Django [Service](https://kubernetes.io/docs/concepts/services-networking/service/) resource created
  - `metadata.name` = `django`
  - `spec.selector.name` = `django`
  - `spec.ports['name']` = http
  - `spec.ports['port']` = 80
  - `spec.ports['protocol']` = TCP
  - `spec.ports['targetPort']` = 80
```
your outcome should be something like the following example below
```
apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.metadata.namespace }}
  name: django-service
spec:
  selector:
    app: {{ .Values.django.name }}
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 80
```
now that all of your template files are created make sure you create your chart .yaml file and make sure it looks like the following example below
```
name: stud3-djangocomex
version: 1.0.0
```

once that chart.yaml is created create your values.yaml and input the following based off of the template
```
- PVC resource created
  - `spec.resources.requests.storage`
- Secret resource created
  - `data.POSTGRES_PASSWORD`
- Postgres Service resource created
  - NO MINIMUM TEMPLATING REQUIREMENTS
- Statefulset resource created
  - `spec.template.spec.containers['image']`
- Deployment resource created
  - `spec.replicas`
  - `spec.template.spec.containers['image']`
- Ingress resource created
  - `spec.rules['host']`
- Django Service resource created
  - NO MINIMUM TEMPLATING REQUIREMENTS

```
the outcome of your values.yaml should look like the example below
```
metadata:
  namespace: django-system
  
django:
  image: harbor.vip.ark1.soroc.mil/stud3/studybud@sha256:23eb42f58d9eb0d68c8488254721dd9fb76488c846646cf45f665fc4e813502d
  name: django
  replicas: 1
  host: django.vip.stud3.soroc.mil

db:
  image: harbor.vip.ark1.soroc.mil/stud3/django-postgres@sha256:7ca2141366e808bd0cbd78218b1d418c7c4b339f90c70d8eceac39b166505438
  name: django-db
  storage: 20Gi

secret:
  pass: UEBzc3cwcmQ=
```


## DEPLOYING A DjANGO APP USING HARBOR HELM CHART

first create a namespace on your server that it is going to use
```
kubectl create namespace <django-app>
```
log in to rancher

go to your repositories on rancher and navigate to your harbor repository that already has your uploaded helm chart on it

from that point click on the chart Django-app and name it studybud

then click next
and finally install

on your server node type the following command

```
watch kubectl get all -n django-app
```
the output should look something like this
```
NAME                          READY   STATUS    RESTARTS      AGE
pod/django-698d9c54dc-6qsx7   1/1     Running   2 (21s ago)   33s
pod/django-db-0               1/1     Running   0             33s

NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/django      ClusterIP   10.43.222.20   <none>        80/TCP     33s
service/django-db   ClusterIP   10.43.50.180   <none>        5432/TCP   33s

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/django   1/1     1            1           33s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/django-698d9c54dc   1         1         1       33s

```

