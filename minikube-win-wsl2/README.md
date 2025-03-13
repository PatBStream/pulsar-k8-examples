- [Intro](#intro)
  - [Assumptions](#assumptions)
- [Setup Minikube](#setup-minikube)
- [Setup Helm](#setup-helm)
- [Add Helm Repo for Pulsar](#add-helm-repo-for-pulsar)
- [Install Pulsar using Helm (no security)](#install-pulsar-using-helm-no-security)
- [Install Pulsar using Helm (with security, self-signed certs)](#install-pulsar-using-helm-with-security-self-signed-certs)
- [Cleanup and remove Pulsar and Minikube](#cleanup-and-remove-pulsar-and-minikube)
- [Demos and Example with Pulsar on Minikube](#demos-and-example-with-pulsar-on-minikube)
# Intro
This doc will describe the setup and install of **Apache Pulsar** on Windows 11 with Windows Subsystem for Linux (WSL2), using **"[minikube](https://minikube.sigs.k8s.io/docs/start/)"** as the K8s env.

After the installation, goto [Running Pulsar](RUN-README.md) for more examples, demos, and troubleshooting items.

**OR**
Setup Pulsar Cluster with security enabled.  Follow the steps below, then goto [Running Pulsar with security](RUN-SECURE-README.md) for further details on access and examples.

## Assumptions

* Linux OS, like Windows 11 with WSL2 installed and running.
* Docker or DockerDesktop for Windows with WSL2 extension, open WSL2 Ubuntu command window:
```
mylaptop@DESKTOP:~$ docker version
Client: Docker Engine - Community
 Cloud integration: v1.0.29
 Version:           20.10.21
 API version:       1.41
 Go version:        go1.18.7
 Git commit:        baeda1f
 Built:             Tue Oct 25 18:02:28 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true
```
* Access to the Internet, no firewall blocks to known repos like GitHub.com

# Setup Minikube
Download minikube for Linux:
```
** Intel Processor Arch **
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
cd /usr/local/bin
sudo ln -s ~/minikube/minikube-linux-amd64 minikube

OR 
** ARM64 Arch  **
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-arm64
cd /usr/local/bin
sudo ln -s ~/minikube/minikube-linux-arm64 minikube
sudo chmod +x ~/minikube/minikube-linux-arm64
```
Run minikube with parameters:
```
minikube start --memory=8192 --cpus=4 --kubernetes-version=v1.25.1
mylaptop@DESKTOP:~$ minikube start --memory=8192 --cpus=4
😄  minikube v1.23.0 on Ubuntu 20.04 (amd64)
🆕  Kubernetes 1.25.1 is now available. If you would like to upgrade, specify: --kubernetes-version=v1.25.3
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🏃  Updating the running docker "minikube" container ...
🐳  Preparing Kubernetes v1.23.1 on Docker 20.10.20 ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass

❗  /usr/local/bin/kubectl is version 1.25.2, which may have incompatibilities with Kubernetes 1.23.1.
    ▪ Want kubectl v1.25.1? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

# Setup Helm
Download and install Helm:
```
** Intel Processor Arch **
wget https://get.helm.sh/helm-v3.10.2-linux-amd64.tar.gz
tar xvf helm-v3.10.2-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm

Or 

** ARM64 Arch **
wget https://get.helm.sh/helm-v3.10.2-linux-arm64.tar.gz
tar xvf helm-v3.10.2-linux-arm64.tar.gz
sudo mv linux-arm64/helm /usr/local/bin/helm

```

Run Helm to verify installation:
```
mylaptop@DESKTOP:~$ helm version
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/pat/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/pat/.kube/config
version.BuildInfo{Version:"v3.10.2", GitCommit:"50f003e5ee8704ec937a756c646870227d7c8b58", GitTreeState:"clean", GoVersion:"go1.18.8"}
```
# Add Helm Repo for Pulsar
This demo/example will use open source [Apache Pulsar](https://pulsar.apache.org/docs/4.0.x/getting-started-helm/)
```
mylaptop@DESKTOP:~$ helm repo add apache https://pulsar.apache.org/charts
mylaptop@DESKTOP:~$ helm repo update
mylaptop@DESKTOP:~$ helm repo list
NAME            URL
apache  https://pulsar.apache.org/charts

```
# Install Pulsar using Helm (no security)
To install Pulsar using Helm, best to "git clone" or download the Pulsar Helm Chart repo.
[https://github.com/apache/pulsar-helm-chart]

We'll use the scripts from this repo to setup and install Pulsar K8 namespace and JWT tokens.

## Download Pulsar Helm Charts and Scripts
Download file [dev-values-pulsar-4.0.2.yaml](helm-values/dev-values-pulsar-4.0.2.yaml) to use Pulsar 4.0.2 and how to change Pulsar versions within the Helm **values** file.

Next, run the "prepare_helm_release.sh" script from the "pulsar-helm-chart" repo, to setup the K8 namespace and tokens:
```
mylaptop@DESKTOP:~$ ./pulsar-helm-chart/scripts/pulsar/prepare_helm_release.sh \
    -n pulsar \
    -k pulsar \
    -c
namespace/pulsar created
generate the token keys for the pulsar cluster
secret/pulsar-token-asymmetric-key created
generate the tokens for the super-users: proxy-admin,broker-admin,admin
generate the token for proxy-admin
pulsar-token-asymmetric-key
secret/pulsar-token-proxy-admin created
generate the token for broker-admin
pulsar-token-asymmetric-key
secret/pulsar-token-broker-admin created
generate the token for admin
pulsar-token-asymmetric-key
secret/pulsar-token-admin created
-------------------------------------

The jwt token secret keys are generated under:
    - 'pulsar-token-asymmetric-key'

The jwt tokens for superusers are generated and stored as below:
    - 'proxy-admin':secret('pulsar-token-proxy-admin')
    - 'broker-admin':secret('pulsar-token-broker-admin')
    - 'admin':secret('pulsar-token-admin')
```

Next, run Helm with the local dev values.yaml file:
```
mylaptop@DESKTOP:~$ helm install pulsar apache/pulsar --namespace pulsar -f dev-values-pulsar-4.0.2.yaml
NAME: pulsar
LAST DEPLOYED: Wed Mar 12 13:09:19 2025
NAMESPACE: pulsar
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing Apache Pulsar Helm chart version 3.9.0.
```
# Install Pulsar using Helm (with security, self-signed certs)
Download file [dev-auth-selfsign-tls-4.0.2.yaml](helm-values/dev-auth-selfsign-tls-4.0.2.yaml) to use Pulsar 4.0.2 with TLS enabled using self-signed certs.

**IMPORTANT** Must load the Cert-Manager CRDs before running Helm install
After Minikube cluster is running normally, enter these command to setup Pulsar with security enabled.
```
mylaptop@DESKTOP:~$ ./pulsar-helm-chart/scripts/cert-manager/install-cert-manager.sh
mylaptop@DESKTOP:~$ helm install pulsar apache/pulsar --namespace pulsar -f dev-auth-selfsign-tls-4.0.2.yaml
```
# Cleanup and remove Pulsar and Minikube  
To cleanup and remove the Pulsar Cluster and Minikube env, run the command:
```
mylaptop@DESKTOP:~$ minikube delete --all
```
# Demos and Example with Pulsar on Minikube
After the installation steps, goto [Running Pulsar](RUN-README.md) for more examples, demos, and troubleshooting items.  
[Running Pulsar with Demos and Examples](RUN-README.md)  
  
Setup Pulsar Cluster with security enabled.  Follow the install steps above, then goto [Running Pulsar with security](RUN-SECURE-README.md) for further details on access and examples.  

[Running Pulsar with security](RUN-SECURE-README.md)  

