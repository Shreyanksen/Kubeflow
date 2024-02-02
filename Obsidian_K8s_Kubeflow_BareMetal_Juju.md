# Kubeflow with Kubernetes Cluster using Juju on Bare Metal

### Prerequisite:

Ubuntu 20.04(focal) or later

- At least 4 cores, 32GB RAM and 50GB of disk space available

- Internet for downloading the required snaps and charms

Charmed Kubeflow requires your Kubernetes instance to:

- Kuberenetes , kubeadm and kubelet be version 1.22

- Have a (default) storage class configured

- Have dns configured for accessing the dashboard, have a LoadBalancer and Ingress


**Step 1 - Update Ubuntu**

Always recommended updating the system packages.

> Perform them on **All Nodes**

```
sudo apt update && sudo apt upgrade -y
```

---------------------------------------------------------------------

**Step 2 - Install Docker**

Kubernetes requires an existing Docker installation.

> Perform them on **All Nodes**

Install Docker with the command:

```bash
sudo apt install docker.io -y
```

- Check the installation (and version) by entering the following:

```bash
docker --version
```

---------------------------------------------------------------------

**Step 3 - Start and Enable Docker**

> Perform them on **all Nodes**

- Set Docker to launch at boot by entering the following:

```bash
sudo systemctl enable docker
```

```bash
sudo systemctl start docker
```

- Verify Docker is running:
```bash
sudo systemctl status docker
```

```bash
● docker.service - Docker Application Container Engine

Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)

Active: active (running) since Sat 2020-10-20 13:53:02 -03; 2h 54min ago

TriggeredBy: ● docker.socket

Docs: https://docs.docker.com

Main PID: 2282 (dockerd)

Tasks: 21

Memory: 146.3M

CGroup: /system.slice/docker.service

└─2282 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

- **To start Docker if it’s not running:**

```bash
sudo systemctl start docker
```
-----------------------------------------------------------------
**Step 4 - Install Kubernetes**

As we are downloading Kubernetes from a non-standard repository, it is essential to ensure that the software is authentic. This is done by adding a subscription key.

> Perform them on **all Nodes**

Enter the following to add a signing key in you on Ubuntu:
```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
```

---------------------------------------------------------------------

**Step 5 - Add Software Repositories**

> Perform them on **all Nodes**

Kubernetes is not included in the default repositories. To add them, enter the following:
```bash
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

---------------------------------------------------------------------

**Step 6 - Kubernetes Installation Tools**

Kubernetes Admin or Kubeadm is a tool that helps initialize a cluster. Kubelet is the work package, which runs on every node and starts containers. The tool gives you command-line access to clusters.

> Perform them on **all Nodes**

***Version used is 1.22.17***

**Install Kubernetes tools with the command:**

```bash
sudo apt-get install kubeadm=1.22\* kubelet=1.22\* kubectl=1.22\* -y
```

```bash
sudo apt-mark hold kubeadm kubelet kubectl
```

Allow the process to complete.

Verify the installation with:

```bash
kubeadm version
```

------------------------------------------------------------------------

**Step 7 - Create daemon.json File for Docker**

After installation of kubeadm, kubelet and kubectl, now you have to make a **daemon.json** file in /etc/docker

> Perform them on **all Nodes**

```bash
sudo nano /etc/docker/daemon.json
```

And there you have to paste this:
```bash
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
```

- Save it and exit

After that you have to perform some service and restart it.

```bash
sudo systemctl enable docker
```

```bash
sudo systemctl daemon-reload
```

```bash
sudo systemctl restart docker
```

```bash
sudo systemctl status kubelet
```

```bash
sudo systemctl start kubelet
```

--------------------------------------------------------------------

**Step 8 - Kubernetes Deployment**

> Perform it on **all Nodes**

Start by disabling the swap memory:
```bash
sudo swapoff -a
```

Also check entries in **/etc/fastab**

- If swap is **on** comment the line to disable it.

---------------------------------------------------------------------

**Step 9 - Assign Unique Hostname for Each Server Node**

Decide which server to set as the master node. Then enter the command:
```bash
sudo hostnamectl set-hostname master-node
```

Next, set a worker node hostname by entering the following on the worker server:
```bash
sudo hostnamectl set-hostname w1
```

If you have additional worker nodes, use this process to set a unique hostname for the rest.

---------------------------------------------------------------------

**Step 10 - Update hosts file**

Finally we need to add all the IP adress of the nodes along with the names assigned in the /etc/hosts file in Every node.

Example:
```bash
cat /etc/hosts
```

```bash
10.0.5.8 master-node

10.0.5.9 w1
```

----------------------------------------------------------------------

> Perform it on only **Master Node**

**Step 11 - Initialize Kubernetes on Master Node**

Switch to the master server node, and enter the following:

Prequisits if possible:

```bash
sudo kubeadm config images pull
```

Now perform this step to create the network for K8s Cluster and make the master node act as a controller node.
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=all
```

- Once this command finishes, it will display a **kubeadm join** message at the end.
```bash
I0113 10:10:33.443886 51200 version.go:255] remote version is much newer: v1.26.0; falling back to: stable-1.22

[init] Using Kubernetes version: v1.22.17

[preflight] Running pre-flight checks

[preflight] Pulling images required for setting up a Kubernetes cluster

[preflight] This might take a minute or two, depending on the speed of your internet connection

[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'

[certs] Using certificateDir folder "/etc/kubernetes/pki"

[certs] Generating "ca" certificate and key

[certs] Generating "apiserver" certificate and key

[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local master-node] and IPs [10.96.0.1 172.16.4.178]

[certs] Generating "apiserver-kubelet-client" certificate and key

[certs] Generating "front-proxy-ca" certificate and key

[certs] Generating "front-proxy-client" certificate and key

[certs] Generating "etcd/ca" certificate and key

[certs] Generating "etcd/server" certificate and key

[certs] etcd/server serving cert is signed for DNS names [localhost master-node] and IPs [172.16.4.178 127.0.0.1 ::1]

[certs] Generating "etcd/peer" certificate and key

[certs] etcd/peer serving cert is signed for DNS names [localhost master-node] and IPs [172.16.4.178 127.0.0.1 ::1]

[certs] Generating "etcd/healthcheck-client" certificate and key

[certs] Generating "apiserver-etcd-client" certificate and key

[certs] Generating "sa" key and public key

[kubeconfig] Using kubeconfig folder "/etc/kubernetes"

[kubeconfig] Writing "admin.conf" kubeconfig file

[kubeconfig] Writing "kubelet.conf" kubeconfig file

[kubeconfig] Writing "controller-manager.conf" kubeconfig file

[kubeconfig] Writing "scheduler.conf" kubeconfig file

[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"

[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"

[kubelet-start] Starting the kubelet

[control-plane] Using manifest folder "/etc/kubernetes/manifests"

[control-plane] Creating static Pod manifest for "kube-apiserver"

[control-plane] Creating static Pod manifest for "kube-controller-manager"

[control-plane] Creating static Pod manifest for "kube-scheduler"

[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"

[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s

[apiclient] All control plane components are healthy after 12.505031 seconds

[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace

[kubelet] Creating a ConfigMap "kubelet-config-1.22" in namespace kube-system with the configuration for the kubelets in the cluster

[upload-certs] Skipping phase. Please see --upload-certs

[mark-control-plane] Marking the node master-node as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]

[mark-control-plane] Marking the node master-node as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]

[bootstrap-token] Using token: 673zsr.n63vmdcz4t04uxmn

[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles

[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes

[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials

[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token

[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster

[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace

[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key

[addons] Applied essential addon: CoreDNS

[addons] Applied essential addon: kube-proxy


Your Kubernetes control-plane has initialized successfully!


To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube


sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config


sudo chown $(id -u):$(id -g) $HOME/.kube/config


Alternatively, if you are the root user, you can run:

export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.

Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:

https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.5.8:6443 --token e2gn9o.bvpct35bas0mrtr6 --discovery-token-ca-cert-hash sha256:b982d72bc94e95b0633a9697d77420d2849aa93dd8910c557edcc0d4dc637117 
```

- Make a note of the Join command.
```bash
kubeadm join 10.0.5.8:6443 --token e2gn9o.bvpct35bas0mrtr6 --discovery-token-ca-cert-hash sha256:b982d72bc94e95b0633a9697d77420d2849aa93dd8910c557edcc0d4dc637117
```

 - This will be used to join the worker nodes to the cluster, later on.

-----------------------------------------------------------------------

**Step 12 - Next, Create a directory for the cluster:**

> Perform it on only **Master Node**

```bash
mkdir -p $HOME/.kube
```

```bash
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

```bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

-----------------------------------------------------------------------

> Perform only on Master Node

**Step 13 - Deploy Pod Network to Cluster(CNI)**

A Pod Network is a way to allow communication between different nodes in the cluster.
- This tutorial uses the flannel virtual network.

### Fnallel CNI:

Enter the following:
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### Calico CNI:

Enter the following:
```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/canal.yaml
```

Allow the process to complete.

--------------------------------------------------------------

**Step 14 - Verify that everything is running and communicating:**

```bash
kubectl get pods --all-namespaces -o wide
```

_Output_:
```bash
NAME                                       READY   STATUS    RESTARTS       AGE    IP            NODE          NOMINATED NODE   READINESS GATES
calico-kube-controllers-68d86f8988-qt5cq   1/1     Running   0              2d1h   10.244.0.3    master-node   <none>           <none>
canal-mx9ws                                2/2     Running   0              3d1h   10.0.5.8      master-node   <none>           <none>
canal-q7shl                                2/2     Running   2 (2d1h ago)   3d1h   10.0.5.9      w1            <none>           <none>
coredns-7bdbbf6bf5-4wh5w                   1/1     Running   0              2d1h   10.244.1.89   w1            <none>           <none>
coredns-7bdbbf6bf5-pwltk                   1/1     Running   0              2d1h   10.244.0.2    master-node   <none>           <none>
etcd-master-node                           1/1     Running   0              3d1h   10.0.5.8      master-node   <none>           <none>
kube-apiserver-master-node                 1/1     Running   0              3d1h   10.0.5.8      master-node   <none>           <none>
kube-controller-manager-master-node        1/1     Running   0              3d1h   10.0.5.8      master-node   <none>           <none>
kube-proxy-4wf46                           1/1     Running   1 (2d1h ago)   3d1h   10.0.5.9      w1            <none>           <none>
kube-proxy-9p4m4                           1/1     Running   0              3d1h   10.0.5.8      master-node   <none>           <none>
kube-scheduler-master-node                 1/1     Running   0              3d1h   10.0.5.8      master-node   <none>           <none>
```

-----------------------------------------------------------------------

> Perform on worker nodes Only

**Step 15 - Join the Worker Node to Cluster**

As indicated in Step 11, you can enter the kubeadm join command on **each worker nodes** to connect it to the cluster.

Switch to the w1, w2 etc respectively and enter the command you noted from Step 11:

use **sudo** with this command

```bash
sudo kubeadm join --discovery-token abcdef.1234567890abcdef --discovery token-ca-cert-hash sha256:1234..cdef 1.2.3.4:6443
```

also use, this flag if required:

```bash
--ignore-preflight-errors=all
```

Switch to the master server, and enter:

```bash
kubectl get nodes -o wide
```

```bash
NAME          STATUS   ROLES                  AGE    VERSION    INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master-node   Ready    control-plane,master   3d1h   v1.22.17   10.0.5.8      <none>        Ubuntu 20.04.6 LTS   5.4.0-146-generic   docker://20.10.21
w1            Ready    <none>                 3d1h   v1.22.17   10.0.5.9      <none>        Ubuntu 20.04.6 LTS   5.4.0-146-generic   docker://20.10.21
```

----------------------------------------------------------------------

> Perform it on only **Master Node**

**Step 16 - Provision storage class:**
```bash
kubectl get sc
```

_Output:_
```bash
ubuntu@master-node:~$ kubectl get sc
No resources found
```

Now apply the storage class file for kubernetes :
```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```

_Output_:
```bash
namespace/local-path-storage created

serviceaccount/local-path-provisioner-service-account created

clusterrole.rbac.authorization.k8s.io/local-path-provisioner-role created

clusterrolebinding.rbac.authorization.k8s.io/local-path-provisioner-bind created

deployment.apps/local-path-provisioner created

storageclass.storage.k8s.io/local-path created

configmap/local-path-config created
```

```bash
kubectl -n local-path-storage get pod -o wide
```

_Output_:
```bash
NAME                                      READY   STATUS    RESTARTS       AGE    IP             NODE   NOMINATED NODE   READINESS GATES
local-path-provisioner-849cb58dff-smldz   1/1     Running   2 (2d1h ago)   2d1h   10.244.1.108   w1     <none>           <none>
```

Patch the storage class:
```bash
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

Output:
```bash
kubectl get sc
```

```bash
NAME                   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  3d1h
```

------------------------------------------------------------------
> Perform it on only **Master Node**

**Step 17 - Configure MetaLLB for kubernetes cluster deployed on Bare Metal**

- Apply the MetaLLB Manifest file :
```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.9/config/manifests/metallb-native.yaml
```

- Check the Internal IP of the cluster Deployed on:
```bash
kubectl get nodes -o wide
```

```bash
NAME          STATUS   ROLES                  AGE    VERSION    INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master-node   Ready    control-plane,master   3d1h   v1.22.17   10.0.5.8      <none>        Ubuntu 20.04.6 LTS   5.4.0-146-generic   docker://20.10.21
w1            Ready    <none>                 3d1h   v1.22.17   10.0.5.9      <none>        Ubuntu 20.04.6 LTS   5.4.0-146-generic   docker://20.10.21
```

<p> In this case the IP range is 10.0.4.0/21, So we need to have a set of IP range for the Load Balancer and that should be available within the same network. </p>

<p>I am going to use IP range from 10.0.5.11 to 10.0.5.20. You are free to use any range as long as you have free IP's to use as a load Balancer within the Network. </p>

- Create File for Specifying the IP Pool :
```
nano ippool.yaml
```

- File content :
```bash
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 10.0.5.11-10.0.5.20
```

- Apply the file :
```bash
kubectl create -f ippool.yaml
```

- Check the applied file pod:
```bash
kubectl get ipaddresspool.metallb.io -A
```

- Output:
```bash
NAMESPACE        NAME         AUTO ASSIGN   AVOID BUGGY IPS   ADDRESSES
metallb-system   first-pool   true          false             ["10.0.5.11-10.0.5.20"]
```

- Decribe the pod:
```bash
kubectl describe ipaddresspool.metallb.io first-pool -n metallb-system
```

```bash
Name:         first-pool
Namespace:    metallb-system
Labels:       <none>
Annotations:  <none>
API Version:  metallb.io/v1beta1
Kind:         IPAddressPool
Metadata:
  Creation Timestamp:  2023-04-12T05:43:37Z
  Generation:          1
  Managed Fields:
    API Version:  metallb.io/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        .:
        f:addresses:
        f:autoAssign:
        f:avoidBuggyIPs:
    Manager:         kubectl-create
    Operation:       Update
    Time:            2023-04-12T05:43:37Z
  Resource Version:  2150
  UID:               e08b8f74-6b17-4970-90b6-667f9e05d280
Spec:
  Addresses:
    10.0.5.11-10.0.5.20
  Auto Assign:       true
  Avoid Buggy I Ps:  false
Events:              <none>
```

- Layer 2 metaLLB system
```bash
nano layer2-metallb.yaml
```

Paste this file content :
```bash
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
spec:
  ipAddressPools:
  - first-pool
```

Apply the File :
```bash
kubectl create -f layer2-metallb.yaml
```

Check the Pod :
```bash
kubectl get l2advertisements.metallb.io -A
```

_Output_ :

```bash
NAMESPACE        NAME      IPADDRESSPOOLS   IPADDRESSPOOL SELECTORS   INTERFACES
metallb-system   example   ["first-pool"] 
```

Describe the pod for more details:
```bash
kubectl describe l2advertisements.metallb.io example -n metallb-system
```

_Output_:

```bash
Name:         example
Namespace:    metallb-system
Labels:       <none>
Annotations:  <none>
API Version:  metallb.io/v1beta1
Kind:         L2Advertisement
Metadata:
  Creation Timestamp:  2023-04-12T05:49:46Z
  Generation:          1
  Managed Fields:
    API Version:  metallb.io/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        .:
        f:ipAddressPools:
    Manager:         kubectl-create
    Operation:       Update
    Time:            2023-04-12T05:49:46Z
  Resource Version:  2627
  UID:               cc07917a-e742-48fe-8ec8-6580fda19aa1
Spec:
  Ip Address Pools:
    first-pool
Events:  <none>
```

- Finally Check all MetaLLB Pods :
```bash
kubectl -n metallb-system get all
```

_Output_:

```bash
NAME                              READY   STATUS    RESTARTS       AGE
pod/controller-5684477f66-txwv6   1/1     Running   4 (2d1h ago)   2d1h
pod/speaker-b9hx4                 1/1     Running   2 (2d1h ago)   3d1h
pod/speaker-c7sqh                 1/1     Running   0              3d1h

NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/webhook-service   ClusterIP   10.110.80.86   <none>        443/TCP   3d1h

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/speaker   2         2         2       2            2           kubernetes.io/os=linux   3d1h

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           3d1h

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-5684477f66   1         1         1       3d1h
```

- Test MetaLLB with Nginx Pod:

- Create a Nginx Pod with LoadBalancer as a service to check if accesible outside the cluster

```bash
kubectl create deploy nginx --image nginx
```

Add service to the Pod:

```bash
kubectl expose deploy nginx --port 80 --type LoadBalancer
```

Test all Pods:

```bash
kubectl get all
```

```bash
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-6799fc88d8-tq5dn   1/1     Running   0          2d1h

NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP        3d1h
service/nginx        LoadBalancer   10.109.95.84   10.0.5.11     80:32282/TCP   3d

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           3d

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-6799fc88d8   1         1         1       3d
```

<p>Here we can see the External_IP assigned as "10.0.5.11" which is the First IP of the MetaLLB We specified while creating th ippool yaml file, similarly other services will get the IP from the LB and the range we specifed.</p>

Test the IP :

```
curl 10.0.5.11
```

_Output_:

```bash
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

- **Or We can Also use a web browser and open the IP and we should see the Nginx default Page.**
---------------------------------------------------------------------

> Perform it on only **Master Node**

**Step 18 - Charmed Kubeflow Installation using Juju**

- Install the Juju CLI client via snap:
    ```bash
    sudo snap install juju --classic
    ``` 
    
    If the installation was successful, you will see a message similar to the one below:
    
    ```bash
    juju 2.9.42 from Canonical✓ installed
    ```

Register your cloud with Juju:

We have to create ourcloud. Now, let’s register it with Juju!

This always involves the same basic logic:

Add your cloud to Juju.

Before that it's good to check what clouds, controllers and models basically some common commands that would be useful during this project and what do we have by default so that it would be easy to understand the next steps:

```bash
juju models
```

```bash
juju controllers
```

```bash
juju whoami
```

```bash
juju add-k8s myk8scloud
```

```bash
juju status
```

```bash
juju clouds
```

```bash
Only clouds with registered credentials are shown.
There are more clouds, use --all to see them.


Clouds available on the client:
Cloud                        Regions  Default    Type  Credentials  Source    Description
kubernetes-admin@kubernetes  0                   k8s   0            built-in  A local Kubernetes context
localhost                    1        localhost  lxd   0            built-in  LXD Container Hypervisor
```

**Start Deploying resources for Juju:**
 - Here we have specified our cloud name as myk8scloud
```bash
juju add-k8s myk8scloud
```

```bash
juju clouds
```
Output:
```bash
Clouds available on the client:
Cloud                        Regions  Default    Type  Credentials  Source    Description
kubernetes-admin@kubernetes  0                   k8s   0            built-in  A local Kubernetes context
localhost                    1        localhost  lxd   0            built-in  LXD Container Hypervisor
myk8scloud                   0                   k8s   1            local     A Kubernetes Cluster
```

- Now we will bootstrap our cloud, we have named our cloud-controller as kubeflow-controller:
    ```bash
    juju bootstrap myk8scloud kubeflow-controller
    ```
    Output:
    ```bash
    Clouds available on the controller:
    Cloud       Regions  Default  Type
    myk8scloud  0                 k8s  

    Clouds available on the client:
    Cloud                        Regions  Default    Type  Credentials  Source    Description
    kubernetes-admin@kubernetes  0                   k8s   0            built-in  A local Kubernetes context
    localhost                    1        localhost  lxd   0            built-in  LXD Container Hypervisor
    myk8scloud                   0                   k8s   1            local     A Kubernetes Cluster
    ```

- Now we will add our model "Kubeflow"
    ```bash
    juju add-model kubeflow
    ```
    
    ```bash
    juju models
    ```
    Output:
    ```bash
    Controller: kubeflow-controller

    Model       Cloud/Region  Type        Status     Units  Access  Last connection
    controller  myk8scloud    kubernetes  available  -       admin  just now
    kubeflow*   myk8scloud    kubernetes  available  35      admin  5 hours ago
    ```
#### Check Controllers:
```bash
juju controllers
```

```bash
Use --refresh option with this command to see the latest information.

Controller            Model     User   Access     Cloud/Region  Models  Nodes  HA  Version
kubeflow-controller*  kubeflow  admin  superuser  myk8scloud         2      1   -  2.9.42  
```
Now deploy kubeflow :

```bash
juju deploy kubeflow --trust
```
> The above step can take upto 90 minutes to get Deployed Fully.
<br>
Let the services get deployed and meanwhile we can see them starting up in real time with :

```bash
watch -c juju status --color
```
and 

```bash
watch kubectl get pods -n kubeflow
```

- Check the istio-ingressgateway-workload, that is the IP where the Kubeflow dashboard will be configured, and also the IP for Load Balancer.

    ```bash
	kubectl -n kubeflow get svc istio-ingressgateway-workload
    ```
    
```bash
NAME                            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
istio-ingressgateway-workload   LoadBalancer   10.109.72.157   10.0.5.12     80:30472/TCP,443:31143/TCP   2d5h
```
- Now some components will need to be configured with the URL. For authentication and allowing access to the dashboard service.It is configured with Juju using the following commands: 

	```bash
	juju config dex-auth public-url=http://10.0.5.12
	```
    
    ```bash
    juju config oidc-gatekeeper public-url=http://10.0.5.12
    ```
- Now enable simple authentication, and set a username and password for your Kubeflow deployment, run the following commands:

    **Note :** Feel free to use any username & password.
    ```bash
    juju config dex-auth static-username=admin
    ```
    
    ```bash
    juju config dex-auth static-password=admin
    ```
- Log in to Charmed Kubeflow

Open on browser or try locally on the master node web browser.

http://10.0.5.12


- Login page :
![[pic1.png]]
- You should now see the Kubeflow “Welcome” page:
![[pic2.png]]
- Click on the “Start Setup” button. On the next screen you will be asked to create a namespace. This is just a way of keeping all the files and settings from one project in a single, easy-to-access place. You can choose any name you like :
![[pic3.png]]
- Once you click on the “Finish” button, the Dashboard will be  displayed! 
![[pic4.png]]


________________________________________________________

  
References :

<ol>

<li>Kubernetes Installation :</li> <br>

<li> Storage Class:</li>

- https://www.youtube.com/watch?v=8BSbpJc8FTw

<br>

<li> MetaLLb : </li>

- https://metallb.universe.tf/

- https://metallb.universe.tf/installation/

- https://metallb.universe.tf/configuration/

- https://www.youtube.com/watch?v=LMOYOtzpoXg

<br>

<li>Kubeflow:</li>

- https://www.kubeflow.org/docs/started/installing-kubeflow/
- https://github.com/kubeflow/manifests

<br>

<li> Juju </li>

- https://charmed-kubeflow.io/
- https://charmhub.io/kubeflow
- https://charmed-kubeflow.io/docs/install 
- https://charmed-kubeflow.io/docs/get-started-with-charmed-kubeflow
- https://juju.is/docs/olm/get-started-with-juju
- https://juju.is/docs/olm/juju-add-k8s
- https://juju.is/docs/olm/juju-add-cloud
- https://juju.is/docs/olm/manage-models
- https://charmed-kubeflow.io/docs/kubeflow-bundle
- https://charmed-kubeflow.io/docs/troubleshooting
- https://github.com/kubeflow/manifests/issues/974

</ol>