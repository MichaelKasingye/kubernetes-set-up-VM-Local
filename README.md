#### Resource  DigitalOcean K8s- https://www.youtube.com/watch?v=MpovOI5eK58  - digital ocean set up
#### Resource  metallb- [https://www.youtube.com/watch?v=MpovOI5eK58](https://www.youtube.com/watch?v=VHJ6qM9EVbs
)  - digital ocean set up


https://www.youtube.com/watch?v=VHJ6qM9EVbs

#### [pullImage secrets](https://www.youtube.com/watch?v=XE76fyM9XyE)


# KUBERNETES SET UP TUTORIAL

Kubernetes Use context, ngnix-ingreress, cert manager on Ubuntu VM
Install docker desktop on your local machine.

``https://docs.docker.com/desktop/install/ubuntu/``

It comes along with kubernetes. Enable Kubernetes after installation.

Install Micro K8s on your VM

``https://microk8s.io/docs/getting-started``

Set up kubernetes context so that your local machine can have access to the remote VM’s Kubernetes cluster.

## CONTEXTS <br/>
It essentially defines the execution environment for your Kubernetes operations. 

Utilise;

```bash
kubectl config view --raw
```

The kubectl config view --raw command outputs the raw Kubernetes configuration as it is stored on your system. This includes information about clusters, contexts, users, and other settings that kubectl uses to interact with Kubernetes clusters.

you will get somethihg like this;
```yaml 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: <base64-encoded-cert-data>
    server: https://cluster-api.example.com
  name: my-cluster
contexts:
- context:
    cluster: my-cluster
    user: my-user
  name: my-context
current-context: my-context
kind: Config
preferences: {}
users:
- name: my-user
  user:
    client-certificate-data: <base64-encoded-client-cert-data>
    client-key-data: <base64-encoded-client-key-data>
```

Edit the above respectively to your context in your local contex. but first access the config file 
```bash
cd ~/.kube
```  
then
 ```bash
 nano config
```

##### Note
Remember to edit this;

```yaml
clusters:
- cluster:
    certificate-authority-data: <base64-encoded-cert-data>
    server: https://cluster-api.example.com
  name: my-cluster
```
to:

`` server: https://102.218.90.89:16443 `` change to the server IP of the VM howewver digital Ocean has its own IP it uses.

```yaml
clusters:
- cluster:
    certificate-authority-data: <base64-encoded-cert-data>
    server: https://102.218.90.60:16443
  name: my-cluster
```

View the contexts:
```bash
kubectl config get-contexts
```

you get:
```
CURRENT   NAME                              CLUSTER                                      AUTHINFO                                           NAMESPACE
*          docker-desktop                    docker-desktop                               docker-desktop                                     
           my-user                           microk8s-cluster                             admin                                              
```

switch context to have access to the VM ``my-user`` kubernetes:
```bash
kubectl config use-context my-user
```
you get:<br/>
``Switched to context "my-user".``

now check the current context you are in;
```bash
kubectl config current-context
```
you get:<br/>
``my-user``

OK, given that we are done with that. Let's set up a load balancer solution using metallb.

## METALLB

### NOTE
When deploying on DigitalOcean you will not need METALLB site it has a loadbalancing solution. 

MetalLB (Metal Load Balancer) is an open-source, software-based load balancer designed to work with Kubernetes. Kubernetes is a popular container orchestration platform that automates the deployment, scaling, and management of containerized applications. While Kubernetes provides many features for managing containers, it doesn't include an integrated solution for load balancing external traffic to services within the cluster.

MetalLB fills this gap by enabling a network load balancer for Kubernetes clusters that can distribute incoming traffic to services running within the cluster. It allows you to expose services externally using standard IP protocols, such as TCP and UDP.

https://metallb.universe.tf/installation/

#### Installation by manifest
To install MetalLB, apply the manifest:

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
```
Create a layer 2 yaml confifugration like this;

For example, the following configuration gives MetalLB control over IPs from 192.168.1.240 to 192.168.1.250, and configures Layer 2 mode:

```bash
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 142.93.222.42-142.93.222.47
```

use this to see the IP ranges for your cluster:
```bash
kubectl get nodes -o wide
```
you get;

```
NAME                   STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP     OS-IMAGE                         KERNEL-VERSION   CONTAINER-RUNTIME
my-user                Ready    <none>   42d   v1.28.2   10.122.0.2    142.93.222.42   Debian GNU/Linux 12 (bookworm)   6.1.0-12-amd64   containerd://1.6.22
```
 Yours could be different. And will use the external IP.

## ADD A PROJECT RESOURCE
create a folder called quote and in it create a yaml file called ``deployment.yaml`` 

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    name: quote-backend
  name: quote-backend
  namespace: develop
spec:
  replicas: 1
  selector:
    matchLabels:
      app: quote-backend
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: quote-backend
    spec:
      containers:
        - image: docker.io/datawire/quote:0.4.1
          imagePullPolicy: IfNotPresent
          name: quote-backend
          resources:
            requests:
              cpu: 50m
              memory: 256Mi
          env:
            - name: NODE_ENV
              value: development
            - name: PORT
              value: '9000'
      imagePullSecrets:
        - name: pullsecret
      restartPolicy: Always


apiVersion: v1
kind: Service
metadata:
  name: quote-backend
  labels:
    service: quote-backend
  namespace: develop
spec:
  selector:
    app: quote-backend
  type: NodePort
  ports:
    - name: http
      protocol: TCP
      port: 9000
      targetPort: 9000
      nodePort: 30030
    - name: https
      protocol: TCP
      port: 9005
      targetPort: 9005
      nodePort: 30031


```

Create the develop namespace:
```bash
kubectl create ns develop
```

Apply the resource like this
```bash
kubectl apply -f quote/deployment.yaml
```
Check if everything is running;

```bash
kubectl -n develop get all
```

you will see something like this;
However yours may show more information like services running in that name space;

```bash
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/quote  1/1       1            1           2d12h

NAME                     DESIRED   CURRENT   READY   AGE
replicaset.apps/quote-6c8f564ff   1         1         1       2d12h
```
## SET UP ARGOCD
ArgoCD is a declarative, GitOps-based continuous delivery tool for Kubernetes. It automates the deployment of applications to Kubernetes clusters, ensuring that the live state of your applications matches the desired state specified in a Git repository. Key features of ArgoCD include: GitOps Integration, Continuous Sync, Real-time Monitoring, Rollbacks, Multi-cluster Support.

#### [ArgoCD LINk](https://argo-cd.readthedocs.io/en/stable/)

### Quick Start

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
This will create a new namespace, argocd, where Argo CD services and application resources will live.

#### Run

```bash
kubectl -n argocd get all
```
To view the running pods and services.

If they are running then add an argocd resource called ``argocd-server-nodeport.yaml`` .
```bash
apiVersion: v1
kind: Service
metadata:
  annotations:
  labels:
    app.kubernetes.io/component: server
    app.kubernetes.io/name: argocd-server
    app.kubernetes.io/part-of: argocd
  name: argocd-server-nodeport
  namespace: argocd
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 8080
    nodePort: 30007
  - name: https
    port: 443
    protocol: TCP
    targetPort: 8080
    nodePort: 30008
  selector:
    app.kubernetes.io/name: argocd-server
  sessionAffinity: None
  type: NodePort
```

and apply it ->  ``kubectl apply -f argocd-server-nodeport.yaml``.

This will run the argoCD GUI on the browser. -> `` IP:30007 ``

#### Initialize password in ArgoCD

``` bash 
argocd admin initial-password -n argocd
```
The initial password for the admin account is auto-generated and stored as clear text in the field password in a secret named argocd-initial-admin-secret.

After getting the password login with that password as ``Admin``. Via the side bar go to the settings sections and update password.

Connect your gitOps repository or repository you want the ArgoCD to monitor to trigger in sync. - [RESOURCE](https://argo-cd.readthedocs.io/en/stable/getting_started/#6-create-an-application-from-a-git-repository) -

#### Run the application in ArgoCD
Create a yaml resource called ``application.yaml`` and add this resource below:

```bash
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myproject-develop
  namespace: argocd
spec:
  destination:
    namespace: default
    server: https://kubernetes.default.svc
  project: default
  source:
    path: environments/staging
    repoURL: https://github.com/myusername/myproject-gitops
    targetRevision: develop
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
and apply it ->  ``kubectl apply -f application.yaml``.

That will trigger the deployment of your application on ArgoCD.

 
## Add INGRESS RESOURCES

##### Ingress-ngnix

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```
##### Update repo

```bash
helm repo update

```
##### Create Ingress-ngnix  namespace
```bash
kubectl create ns ingress-nginx
```
##### Create the ingress-ngnix resource
```bash
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx
```

## Add CERT-MANAGER

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.1/cert-manager.crds.yaml
```
This allows you to easily uninstall and reinstall cert-manager without deleting your installed custom resources.

Check if CRDS have been created.

```bash
kubectl get crds
```

To install the chart with the release name cert-manager:

```bash
## Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io --force-update

## Install the cert-manager helm chart
helm install cert-manager --namespace cert-manager --version v1.15.1 jetstack/cert-manager
```
##### Create cert-manager namespace

```bash
kubectl create namespace cert-manager
```


