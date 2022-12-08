- [Intro](#intro)
  - [Assumptions](#assumptions)
- [Setup GKE Cluster with **gcloud**](#setup-gke-cluster-with-gcloud)
- [Setup Pulsar](#setup-pulsar)
- [Testing Pulsar on GKE](#testing-pulsar-on-gke)
  - [View GKE Pods and Services, External IPs](#view-gke-pods-and-services-external-ips)
  - [Access Pulsar Admin Console](#access-pulsar-admin-console)
  - [Access Pulsar Admin CLI](#access-pulsar-admin-cli)
- [Cleanup and delete Pulsar](#cleanup-and-delete-pulsar)

# Intro
This doc will describe the setup and install of **Apache Pulsar** on Google GKE Kubernetes.

Setup Pulsar Cluster with security enabled.  Follow the steps below, then goto [Running Pulsar with security](RUN-SECURE-README.md) for further details on access and examples.

## Assumptions
Google Cloud account with permissions to setup and manage a GKE cluster.
"gcloud" CLI install and setup to access your GCP account. See install info at: [https://cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install)

**NOTE** - Permissions for GKE require "Owner" in your GCP Project <<project-name>> otherwise some errors may occur while using Helm install, related to RBAC for K8.

```
mylaptop@DESKTOP:~$ helm install pulsar datastax-pulsar/pulsar -f dev-latest-values.yaml
Error: INSTALLATION FAILED: failed pre-install: clusterroles.rbac.authorization.k8s.io "pulsar-kube-prometheus-sta-admission" is forbidden: User "myid@mycompany.com" cannot delete resource "clusterroles" in API group "rbac.authorization.k8s.io" at the cluster scope: requires one of ["container.clusterRoles.delete"] permission(s).
```
If this error above occurs, you do not have full permissions for GCP project and GKE.  See the GCP Console website, IAM screen for your account permissions.  It will need "Owner" permission.

# Setup GKE Cluster with **gcloud**
Below is the gcloud command to create a GKE Cluster named "mygketest" with 7 nodes in GCP Project "myproject".
```
mylaptop@DESKTOP:~$ gcloud container --project "myproject" clusters create "mygketest" --zone "us-central1-c" --no-enable-basic-auth --cluster-version "1.23.12-gke.100" --release-channel "regular" --machine-type "e2-medium" --image-type "COS_CONTAINERD" --disk-type "pd-standard" --disk-size "100" --metadata disable-legacy-endpoints=true --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --max-pods-per-node "110" --num-nodes "7" --logging=SYSTEM,WORKLOAD --monitoring=SYSTEM --enable-ip-alias --network "projects/myproject/global/networks/default" --subnetwork "projects/myproject/regions/us-central1/subnetworks/default" --no-enable-intra-node-visibility --default-max-pods-per-node "110" --no-enable-master-authorized-networks --addons HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver --enable-autoupgrade --enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0 --enable-shielded-nodes --node-locations "us-central1-c"
```
After GKE cluster is created, setup kubectl KUBECONFIG:
```
mylaptop@DESKTOP:~$ gcloud container clusters get-credentials mygketest --zone us-central1-c --project myproject
```
# Setup Pulsar
Setup the Pulsar Cluster using Helm.

**Note** Setup Pulsar with security enabled, since this is a Public Cloud with access from the general Internet.
```
mylaptop@DESKTOP:~$ kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.8.0/cert-manager.crds.yaml
mylaptop@DESKTOP:~$
mylaptop@DESKTOP:~$ helm install pulsar datastax-pulsar/pulsar --namespace pulsar --create-namespace -f dev-auth-selfsign-tls.yaml
```
**NOTE** If you receive error while running "helm install..." it maybe due to permissions as mentioned in Assumptions section.

# Testing Pulsar on GKE

## View GKE Pods and Services, External IPs
```
mylaptop@DESKTOP:~$ kubectl get pods -n pulsar
NAME                                                   READY   STATUS      RESTARTS   AGE
prometheus-pulsar-kube-prometheus-sta-prometheus-0     2/2     Running     0          5m22s
pulsar-adminconsole-64b5474c74-2sbsj                   1/1     Running     0          5m27s
pulsar-autorecovery-7796f898d5-jrjqt                   1/1     Running     0          5m26s
pulsar-bastion-65d5697d64-j9n7t                        1/1     Running     0          5m26s
.
.
.
mylaptop@DESKTOP:~$
mylaptop@DESKTOP:~$ kubectl get services -n pulsar
NAME                                    TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)
                                                      AGE
prometheus-operated                     ClusterIP      None           <none>          9090/TCP
                                                      8m7s
pulsar-adminconsole                     LoadBalancer   NN.NN.NN.NN    NN.NN.NN.NN     80:32426/TCP,443:30310/TCP
.
.
.
mylaptop@DESKTOP:~$
```
## Access Pulsar Admin Console
In a web browser with Internet access, goto the Pulsar Admin Console: https://NN.NN.NN.NN:443  (Note NN.NN.NN.NN is the address from above for "pulsar-adminconsole" external IP.)

## Access Pulsar Admin CLI 
Use kubectl "exec" to login the bastion pod for pulsar-admin access.
```
kubectl exec -it -n pulsar pulsar-bastion-65d5697d64-j9n7t -- bash
I have no name!@pulsar-bastion-65d5697d64-j9n7t:/pulsar$
I have no name!@pulsar-bastion-65d5697d64-j9n7t:/pulsar$ bin/pulsar-admin tenants list
public
pulsar
I have no name!@pulsar-bastion-65d5697d64-j9n7t:/pulsar$
```

# Cleanup and delete Pulsar 
Cleanup and delete GKE Cluster.
```
mylaptop@DESKTOP:~$ gcloud container clusters list
NAME     LOCATION       MASTER_VERSION   MASTER_IP       MACHINE_TYPE  NODE_VERSION     NUM_NODES  STATUS
my-demo  us-central1-a  1.23.8-gke.1900  NN.NN.NN.NN     e2-medium     1.23.8-gke.1900  5          RUNNING

mylaptop@DESKTOP:~$gcloud container clusters delete mydemo --zone us-central1-a --project <GCP Project name>
```