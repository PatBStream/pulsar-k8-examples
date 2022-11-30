- [Intro](#intro)
  - [Assumptions](#assumptions)
- [Setup Minikube](#setup-minikube)
- [Setup Helm](#setup-helm)
- [Add Helm Repo for Pulsar](#add-helm-repo-for-pulsar)
- [Install Pulsar using Helm](#install-pulsar-using-helm)
# Intro
This doc will describe the setup and install of **Apache Pulsar** on Windows 11 with Windows Subsystem for Linux (WSL2), using **"[minikube](https://minikube.sigs.k8s.io/docs/start/)"** as the K8s env.

After the installation steps, goto [Running Pulsar](RUN-README.md) for more examples, demos, and troubleshooting items.

## Assumptions

* Windows 11 with WSL2 installed and running.
* DockerDesktop for Windows with WSL2 extension, open WSL2 Ubuntu command window:
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
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
cd /usr/local/bin
sudo ln -s ~/minikube/minikube-linux-amd64 minikube

Run minikube with parameters:
```
minikube start --memory=8192 --cpus=4 --kubernetes-version=v1.21.1
mylaptop@DESKTOP:~$ minikube start --memory=8192 --cpus=4
😄  minikube v1.28.0 on Ubuntu 20.04 (amd64)
🆕  Kubernetes 1.25.3 is now available. If you would like to upgrade, specify: --kubernetes-version=v1.25.3
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🏃  Updating the running docker "minikube" container ...
🐳  Preparing Kubernetes v1.21.1 on Docker 20.10.20 ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass

❗  /usr/local/bin/kubectl is version 1.25.2, which may have incompatibilities with Kubernetes 1.21.1.
    ▪ Want kubectl v1.21.1? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

# Setup Helm
Download and install Helm:
wget https://get.helm.sh/helm-v3.10.2-linux-amd64.tar.gz
tar xvf helm-v3.10.2-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm

Run Helm to verify installation:
```
mylaptop@DESKTOP:~$ helm version
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/pat/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/pat/.kube/config
version.BuildInfo{Version:"v3.10.2", GitCommit:"50f003e5ee8704ec937a756c646870227d7c8b58", GitTreeState:"clean", GoVersion:"go1.18.8"}
```
# Add Helm Repo for Pulsar
This demo/example will use [DataStax's Luna Streaming](https://docs.datastax.com/en/luna-streaming/docs/quickstart-helm-installs.html)
```
mylaptop@DESKTOP:~$helm repo add datastax-pulsar https://datastax.github.io/pulsar-helm-chart
mylaptop@DESKTOP:~$
mylaptop@DESKTOP:~$ helm repo list
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/pat/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/pat/.kube/config
NAME            URL
datastax-pulsar https://datastax.github.io/pulsar-helm-chart

```
# Install Pulsar using Helm 
Download example Helm chart "dev-values.yaml" file from DataStax at: [https://github.com/datastax/pulsar-helm-chart/tree/master/examples](https://github.com/datastax/pulsar-helm-chart/tree/master/examples)
We'll use this Helm file for our demo.

Next, run Helm with the local dev-values.yaml file:
```
mylaptop@DESKTOP:~$ helm install pulsar -f dev-values.yaml datastax-pulsar/pulsar
```