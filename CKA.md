
- [Certified Kubernetes Administrator Exam Notes](#certified-kubernetes-administrator-exam-notes)
  - [4.0 Understanding API Access and Commands](#40-understanding-api-access-and-commands)
    - [4.2 Core Kubernetes objects](#42-core-kubernetes-objects)
      - [Access](#access)
      - [Checks rights](#checks-rights)
    - [4.3 Options ot explore the API](#43-options-ot-explore-the-api)
      - [Interacting with the API](#interacting-with-the-api)
    - [4.4 Kubectl to Manage API objects.](#44-kubectl-to-manage-api-objects)
    - [4.5 Using YAML files to define API Objects.](#45-using-yaml-files-to-define-api-objects)
    - [4.6 Using Curl to work with API Objects](#46-using-curl-to-work-with-api-objects)
    - [4.6 Understanding other commands](#46-understanding-other-commands)
  - [5.0 Running pods with deployments.](#50-running-pods-with-deployments)
    - [5.1 Understanding Namespaces](#51-understanding-namespaces)
    - [5.2 Managing Pods and Deployments](#52-managing-pods-and-deployments)
    - [5.3 Running Pods by deployments](#53-running-pods-by-deployments)
    - [5.4 Understanding labels and annotations](#54-understanding-labels-and-annotations)
    - [5.5 Managing Rolling updates.](#55-managing-rolling-updates)
    - [5.6 Managing Deployment History](#56-managing-deployment-history)
    - [5.7 Using init containers.](#57-using-init-containers)
    - [5.8 Managing Stateful sets.](#58-managing-stateful-sets)
    - [5.9 Using DaemonSets](#59-using-daemonsets)
  - [6.0 Understanding Kubernetes Storage Options](#60-understanding-kubernetes-storage-options)
    - [6.1 Understanding Kubernetes Storage Options](#61-understanding-kubernetes-storage-options)
    - [6.2 Configuring Pod Volumes](#62-configuring-pod-volumes)
    - [6.3 Configuring Persistent Volume (PV) storage](#63-configuring-persistent-volume-pv-storage)
    - [6.4 Configuring Persistent Volume Claims (PVCs)](#64-configuring-persistent-volume-claims-pvcs)
    - [6.5 Configuring Pod Storage with PV and PVC.](#65-configuring-pod-storage-with-pv-and-pvc)
    - [6.6 Understanding ConfigMaps and Secrets](#66-understanding-configmaps-and-secrets)
    - [6.7 Managing ConfigMaps](#67-managing-configmaps)
    - [6.8 Managing Secrets](#68-managing-secrets)
# Certified Kubernetes Administrator Exam Notes

- Below are notes from the Live Lessons Course by @sandervanvugt for the CKA Exam.

## 4.0 Understanding API Access and Commands

### 4.2 Core Kubernetes objects

- k8s has a collection of APIs
- APIs have versions
- We set the versions in the yaml definitions to specify which API version we want to use.

#### Access

- API access is through RBAC
- RBAC: Account mapped to certificates and associated with a username.
- Defined in `~/.kube/config`

#### Checks rights

- `kubectl auth can-i .....`

- With kube admin credentials answer will always be yes (in most cases).
- A POD is the minimal object that can be managed by k8s.
- Kubernetes manages pods and doesn't manage containers and everything else inside a pod.
- In order to make managing pods easier, Kubernetes is adding the deployment as an API object.
- Deployments:
    - Our application
    - Replica set
- Service:
    - Connect to the deployments and exposes it externally.
- Persistent Volume:
    - Separate API object.
    - Decoupled from the deployment.
    - Created through a Persistent volume claim.

### 4.3 Options ot explore the API

#### Interacting with the API

```bash
# This will show API groups as well as resources within the APIs, so if you wanna know which resources exist
kubectl api-resources
```

```bash
# Can use curl to explore group information.
kubectl proxy
```

```bash
# Show current api versions
kubectl api-versions
```
```bash
# Explore API components. => very important.
kubectl explain
```

### 4.4 Kubectl to Manage API objects.

> Kubectl => default command to interact with the API.

```bash
#Test everything is working
kubectl cluster-info 
```

```bash
# view ~/.kube/config
Kubectl config view
```

```
# Get completion script for Kubectl
kubectl completion bash|zsh
```

### 4.5 Using YAML files to define API Objects.

- apiVersion => api version
- Kind => which API?
- metadata => generic info about the object.
    - name => object identifier
    - namespace => namespace to deploy in.
    - label => identifier
        app: myapp => set key-values for app identifier
- spec => Specification of myapp
    - container
        - name
        - image
        - command

How to get this information?

- by using `kubectl explain <api kind>`

```bash
kubectl explain pod
```
```bash
# info for spec
kubectl explain pod.spec
```
```bash
# info for pod -> spec -> containers
kubectl explain pod.spec.containers
```

### 4.6 Using Curl to work with API Objects

> api-server ---(TLS)---> writes ---(TLS)---> etcd

To use curl we also need to use TLS to communicate with the API server.
For that we need th kube-proxy.
- Usually, curl is not the way to work with the k8s api.
- kubectl reads the information from the kube config so it does not
need the kubeproxy

```bash
#Using simple curl
curl --cert myuser.pem --key myuser-key.pem --cacert /root/myca.pem https://controller:6443/api/v1
```

```bash
# Using kube proxy
kubectl proxy --port=8001 &
curl http://localhost:8001/api/v1/namespace/
```

### 4.6 Understanding other commands

- `etcdctl` command can be used to interrogate and manage the `etcd` database
- `etcdctl2` => interact with v2
- `etcdctl` => version agnostic


## 5.0 Running pods with deployments.

### 5.1 Understanding Namespaces

- Namespaces are a linux kernel feature that is leveraged up to kubernetes level.
- Namespaces implement strict resource separation.
- Resource quota can also be set at the NS level.
- Namespaces can be used to segregate different customer in a shared cluster.

- default => default NS
- kube-node-lease => admin NS for node lease information
- kube-public => World readable NS. Stores Generic info. Usually empty.
- kube-system => Contains all the infrastructure pods => Needed to run kubernetes itself.

`kubectl get ns`
`kubectl get all --all-namespaces`
`kubectl create ns dev`
`kubectl describe ns dev` 

### 5.2 Managing Pods and Deployments

```bash
# Easiest way to create a deployment
kubectl create deployment --image=nginx my-nginx
```

```bash
# Get the deployment in Yaml form 
kubectl get deployments.apps my-nginx -o yaml 
```

```bash
# Fasted way to create a deployment files
kubectl create deployment --dry-run --image=nginx --output=yaml my-nginx-app > nginx-deployment.yaml 
```

### 5.3 Running Pods by deployments

- Deployment scalability via replicaset definition.

```bash
# Increase replicas via kubectl
kubectl scale deployment my-nginx --replicas=3
```

```bash
# Will open the deployment in editor to edit, but we can only change a few things
# like the replica set.
kubectl edit deployments.apps my-nginx-app
```


### 5.4 Understanding labels and annotations

- Labels are what connect applications together.

```bash
# Manual method to set labels
kubectl get deployments --show-labels
```

```bash
# Add another label
kubectl label deployments.apps my-nginx state=demo => adds another label.
```

```bash
# View with tag filer
kubectl get all --selector state=demo
# or
kubectl describe dpl.apps nginx
 
```
**Label is used by the deployment to monitor the availability of the pods**

- We can test it by removing a pod from the running pod in a deployment, and the 
 deployment will create a new one because to it the pod does not exist anymore.

```bash
# Remove label from a pod
kubectl label pod nginx-a123kjb123 run-   => run - remove the label from the pod
# and view it again, the label will be gone and k8s will be bringing up a new one.
kubectl get pods 
``` 

### 5.5 Managing Rolling updates.

> deployment => replicaset => pod1, pod2, pod3

When we do a updates deployment creates a new replicaset. 

> deployment => new-replicaset => pod1, pod2, pod3

and the old pods will be cleaned up.

- Two parameters are used to manage this
  - maxSurge        => How many pods we can have more than the required number of pods.
  - maxUnavailable. => How many pods we can have less than the required number of pods.


```bash
#The parameters are set under the update strategy in a deployment.
kubectl explain deployments.spec.strategy.rollingUpdate
```

- The default is `RollingUpdate`. We can also set to `Recreate`.
- Some application don't support RollingUpdate so we set the strategy to Recreate
  so they never have two containers of the same app running at the same time.

### 5.6 Managing Deployment History

- Managing the rollout update History

```bash
# View rollout history
kubectl rollout -h
```
```bash
# see deployment history of all deployments
kubectl rollout history deployment
```
- Now change the deployment.

```bash
# Update deployment
kubectl edit deployments.apps <definition name>
kubectl rollout history deployment  => we will see the update in the rolling history.
```

```bash
# See the history of a specific deployment and it's specific revisions.
kubectl rollout history deployment rolling-nginx --revision=1 
kubectl rollout history deployment rolling-nginx --revision=2
```

```bash
# And if we want to rollback
kubectl rollout undo deployment rolling-nginx --to-revision=1
```
> We can use record=true with our kubectl commands to get better rollout history

### 5.7 Using init containers.

- Init container is a special case of learning a pod with two containers. 
- Of these two containers, one can be used as an init container. 
- Any init container can be used to prepare something, and this will be 
  done before the main application is started. 
- Before the init container is successfully completed the main 
  application will not be started.
- Init containers are defined using the initContainers field in the pod specification.

```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: initpod
spec:
  containers:
  - name: after-init
    image: busybox
    command: ['sh', '-c', 'echo its running! && sleep 3600'] 
  initContainers: # will start and complete first and then the after-init container will run
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;'] 
```

### 5.8 Managing Stateful sets.


- A StatefulSet is not the most common object but can be useful, it's like a deployment, but it provides guarantees about the ordering and the uniqueness of Pods. 
- StatefulSets maintain a unique identity for each Pod which makes it, so that Pods are not interchangeable and that the identity of the Pods is going to be the same throughout the lifetime of the StatefulSet. 
- Compare that to the deployment where you delete the pod, the deployment is generating the pod again but with a new ID. 
- StatefulSets are valuable if the application has one of the following requirements which can be 
    - unique network identifiers 
    - stable persistent storage or order deployment and scaling 
    - ordered automated rolling updates.
- StatefulSets do come with some limitations as well. So because of these limitations, you should only use StatefulSets if their features are specifically required. To start with 
    - storage must be provided by a persistentVolume. 
    - Deleting a StatefulSet will not delete associated storage. don't know if that is a disadvantage, but it sure is something to consider. 
    - Also, a Headless Service is required to provide network identity for pods in a StatefulSet. So you will always get a service with your StatefulSet. 
    - And to ensure that Pods in a StatefulSet, are terminated properly, the number of Pods should be scaled down to 0 before deleting the StatefulSet. 
      And that's a different approach than deleting a deployment where you can just delete the deployment. 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi

```

> The volumeClaim above needs a persistent volume to be created first before we can actually deploy this.

### 5.9 Using DaemonSets

 - DaemonSet ensures that all or some nodes run a copy of a pod. 
 - So it's a solution to run pods, multiple times, on all of the nodes. And that's particularly useful for surfaces that should be running everywhere.
 - You can find it for kubenetics services, for instance. Kubenetics services that implement monitoring agent or a network agent or login agent or something like that.
 - If you have created a DaemonSet, nice thing about it is that while nodes are added to the cluster, pods are added to them automatically by the DaemonSet. So it does not require any additional action.
 - Deleting a DaemonSet will delete the pods it created as well. 
 - DaemonSets are specific in some situations, for example, you can run a cluster storage Daemon, such as ceph or glusterd on each of the node to provide storage. 
 - They can also be used for log collection, or for monitoring Daemons, such as collectd or Prometheus Node Exporter and others. 

 An example with fluentd.

 ```yaml
 apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

> kubectl get daemonset -n kubesystem => will show us the daemon sets.

## 6.0 Understanding Kubernetes Storage Options

- This section deals with understanding how kubernetes deals with storage.

### 6.1 Understanding Kubernetes Storage Options

Pod => Spec => Volumes (internal. It is ephemeral, will go away when the pod goes away)

- We don't want that to happen. We want the storage to persist.
- For that we have a few options.
  - PVC (Persistent volume claim) => PV (Persistent Volume)  => NFS, Ceph, EBS ....
  - ConfigMap (We can storefiles and vars in the ConfigMap).
  - Secret (Special config map where the variables are secret, base64encoded)

### 6.2 Configuring Pod Volumes

Internally shared volume

```yaml
apiVersion: v1
kind: Service
metadata:
  name: sharedvolume
  labels:
    app: sharedvolume
spec:
  containers:
  - name: centos1
    image: centos:7
    volumeMounts:
      - mountPath: /centos1
        name: test
  - name: centos2
    image: centos:7
    volumeMounts:
      - mountPath: /centos2
        name: test
  volumes:
    - name: test
      emptyDir: {}  # ==> Will just create an empty temporary directory in side the containers.
  ```
- Now to check

```bash
# shell access one of the containers above. By default will shell access the first container
kubectl exec -it sharedvolume -- /bin/bash
# inside the shell
touch centos1/newfile
exit
```
- Now check if the file exists in the second container
```bash
# for the second container
kubectl exec -it sharedvolume -c centos2 -- ls /centos2
newfile # => we see the file exists
```bash
- To see all the types of volumes available
kubectl explain pod.spec.volumes
```

### 6.3 Configuring Persistent Volume (PV) storage

- Persistent volumes deployments create the volume. 
- This will create a persistent volume.
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
  labels:
    app: pv-volume
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mydata"
```
- Then we can do
```bash
# to view the volume
kubectl explain pv.spec.storageClassName
kubectl get pv pv-volume
```
> hostpath is totally used for multinode environments. It's mostly used for testing and single node deployments

- NFS PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
  labels:
    app: pv-volume
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: "/mydata"
    server: myserver
    readonly: false
```

### 6.4 Configuring Persistent Volume Claims (PVCs)

- Once a PV is created it needs to be claimed. For that a persistent volume claim needs to be created.
- The PVC looks for the volume that matches the criteria defined closest to as seen below.
- For example the accessMode must be ReadWriteOnce and storage required is 1Gi so that needs to be satisfied.
- Nothing else is actually needed
```yaml
apiVersion: v1
kind: PersistentVolumeClain
metadata:
  name: pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```bash
# and to view it
kubectl get pvc
```
- Once the volumes have been claimed we can run the command
`kubectl get pv` again and we will see the volumes and which claim they have been claimed by.
- Once the PV has been claimed by the PVCs, the pod can then use it.

### 6.5 Configuring Pod Storage with PV and PVC.

- Inside the pod definition we will first create a volume
- The we will associate the volume with a claim name (myPVC).
- Then we have the PVC itself named myPVC and has an access mode (ReadWriteOnce).
- Then we have a PV with a accessMode (ReadWriteOnce). 
So this is how it's all ties together:
> Pod => volume ==(name)==> ClaimName ==(name)==> PVC ==(accessMode)==> PV (accessMode)

```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: pv-pod
spec:
  volumes:
  - name: pv-storage
  persistentVolumeClaim:
    claimName: pv-claim   # will look externally for a PVC with the name pv-claim. If found it will define it internally as pv-storage
  containers:
  - name: pv-container
    image: nginx
    ports:
      - containerPort: 80
        name: "http-server"
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: pv-storage # will mount the pv-storage (pvclaim) on mounth path.
```

### 6.6 Understanding ConfigMaps and Secrets

- The goal is to decouple things in kubernetes.
- ConfigMaps provide files, contents of directories or variables from outside of a pod.
- So a pod will create references to different things to configMap. For example
- Pod Volumes will reference files, env vars will reference variables....
- And when we want to keep the data in the configmap secret, we use secrets.
- Secrets store base64encode vars.

### 6.7 Managing ConfigMaps

- Lets take nginx config file and name it nginx-customer-config.conf

```lua
server {
    listen       8888;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```

- Lets create a config map from that
```bash
kubectl create cm nginx-cm --from-file nginx-custom-config.conf
```
- And to view it
```bash
kubectl get cm nginx-cm -o yaml
```
- Now we can use this configMap in a pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-cm
  labels:
    role: web
spec:
  containers:
  - name: nginx-cm
    image: nginx
    volumeMounts:
    - name: conf      # Will match the volume.name below
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: conf
    configMap:
      name: nginx-cm  # Needs to be the same as the name of the ConfigMap we definedabove
      items:
      - key: nginx-custom-config.conf
        path: default.conf
```
- The result will be `/etc/nginx/conf.d/default.conf`

- Referencing variable from a configMap

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: test
    image: nginx
    env:
    - name: COLOR  # env var name
      valueFrom:
        configMapKeyRef:
          name: myconfig
          key: color    # get env var values from config map.
  restartPolicy: Never
```

### 6.8 Managing Secrets

- Storing secrets data.
- Used similar to ConfigMaps.
- All values in the secret files must be base64 encoded.

```bash
# to create a secret
kubectl create secret generic mysecrets --from-literal=password=password --from-literal=user=linda
```
- Then to view it
```bash
kubectl get secrets mysecrets -o yaml
```

- Using secrets as volumes:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: test
    image: nginx
    volumeMounts:
    - mountPath: "/topsecret"
      name: secret
  volumes:
  - name: secret
    secret:
      secretName: mysecret
```

result will be two files created
- /topsecret/password
- /topsecret/user

- Using secrets as variables
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql
spec:
  containers:
  - name: mysql
    image: busybox
    env:
    - name: MYSQL_ROOT_PASSWORD  # env var name
      valueFrom:
        configMapKeyRef:
          name: mysql
          key: password    # get env var values from secrets.
```

LAB Question:

Configure a 2Gib persistent storage solution that uses a permanent directory on the host that runs the Pod. 
Configure a Deployment that runs the httpd web server and mounts the storage on /var/www