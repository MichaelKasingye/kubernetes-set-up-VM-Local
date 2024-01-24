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

Edit the above respectively to your context in your local contex.

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
``Switched to context "production-novasuites".``

now check the current context you are in;
```bash
kubectl config current-context
```
you get:<br/>
``my-user``

OK, given that we are done with that. Let's set up a load balancer using metallb.

## METALLB


