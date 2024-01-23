
 KUBERNETES SET UP TUTORIAL

Kubernetes Use context, ngnix-ingreress, cert manager on Ubuntu VM
Install docker desktop on your local machine.

``https://docs.docker.com/desktop/install/ubuntu/``

It comes along with kubernetes. Enable Kubernetes after installation.

Install Micro K8s on your VM

``https://microk8s.io/docs/getting-started``

Set up kubernetes context so that your local machine can have access to the remote VM’s Kubernetes cluster.

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

Edit the above respectively to your context in your..





