- [Intro](#intro)
  - [Highlevel overview of setup](#highlevel-overview-of-setup)
  - [Assumptions](#assumptions)
  - [Install "eksctl" and "aws" Command-line tools](#install-eksctl-and-aws-command-line-tools)
- [Create a EKS cluster](#create-a-eks-cluster)
- [Setup and enable Persistent in EKS](#setup-and-enable-persistent-in-eks)
  - [Check and obtain the OIDC ID from AWS (3 commands below)](#check-and-obtain-the-oidc-id-from-aws-3-commands-below)
  - [Setup Service Account for Persistent](#setup-service-account-for-persistent)
- [Install Pulsar](#install-pulsar)
- [View EKS Pods, Services, and External IPs](#view-eks-pods-services-and-external-ips)
- [Cleanup and delete Pulsar](#cleanup-and-delete-pulsar)
- [Troubleshooting issues](#troubleshooting-issues)

# Intro
This doc will describe the setup and install of **Apache Pulsar** on AWS EKS Kubernetes.

Setup Pulsar Cluster with security enabled.  Follow the steps below, then goto [Running Pulsar with security](RUN-SECURE-README.md) for further details on access and examples.  

General docs and info on AWS EKS is at [https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)  

## Highlevel overview of setup
Using command-line tools "eksctl", "aws", and "kubectl", we will create and demo K8 on AWS.  

* Install required tools
* Create K8 cluster
* Enable "persistent volumes" in EKS
* Install Pulsar on EKS

## Assumptions
This repo/doc requires the proper AWS account with permissions to setup and manage a EKS cluster.  The setup of AWS accounts is beyond the scope of this doc.  Also, we assume "kubectl" is install with the proper versions to interact with K8.  For install of kubectl, see info at [https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)  
  
## Install "eksctl" and "aws" Command-line tools   
This setup/install uses 2 command-line tools for AWS K8 clusters.  

"aws" CLI install and setup to access your AWS account. See install info at: [https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)  

"eksctl" tool with create, update, control, and destroy K8 clusters in AWS.  
To install, see info at [https://eksctl.io/introduction/](https://eksctl.io/introduction/).  Also see eksctl info at [https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)  

# Create a EKS cluster 
After logging into AWS in your environment (), use "eksctl" to create a new K8 cluster.

```
mylaptop@DESKTOP:~$ eksctl create cluster --name mydemo --nodes 4 --kubeconfig=aws-config --node-volume-type=gp2
```
This command will run for several minutes, usually 7 to 10 minutes, depending on your environment.  
**NOTE** This command writes the "kubeconfig" information to a file named "aws-config".  You can remove this parameter, which will then update the default "kubeconfig" file.  

If you use "aws-config" file, you should set environment variable to use it:  
```
mylaptop@DESKTOP:~$ export KUBECONFIG=~/mydir/aws-config
```
# Setup and enable Persistent in EKS
Several steps and commands are required to enable "persistent volumes" in EKS.  Each step is required and must complete successfully.  If not, any "persistent volume claims" (pvc) will show as "pending" in kubectl.  

## Check and obtain the OIDC ID from AWS (3 commands below)
```
mylaptop@DESKTOP:~$ oidc_id=$(aws eks describe-cluster --name mydemo --region us-west-2 --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
mylaptop@DESKTOP:~$
mylaptop@DESKTOP:~$ aws iam list-open-id-connect-providers | grep $oidc_id
mylaptop@DESKTOP:~$
mylaptop@DESKTOP:~$ eksctl utils associate-iam-oidc-provider --cluster pbdemo --approve
```
## Setup Service Account for Persistent
Create a Service Account to allow access and permission to EBS volumes.
```
mylaptop@DESKTOP:~$ eksctl create iamserviceaccount --name ebs-csi-controller-sa --namespace kube-system --cluster mydemo  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy --approve --role-only --role-name AmazonEKS_EBS_CSI_DriverRole
```
Next, create "addon" for EBS.  **NOTE** replace "NNNNNNNNNNNN" with your AWS Account number below.
```
mylaptop@DESKTOP:~$ eksctl create addon --name aws-ebs-csi-driver --cluster mydemo --service-account-role-arn arn:aws:iam::NNNNNNNNNNNN:role/AmazonEKS_EBS_CSI_DriverRole
mylaptop@DESKTOP:~$
mylaptop@DESKTOP:~$ kubectl annotate serviceaccount ebs-csi-controller-sa -n kube-system  eks.amazonaws.com/role-arn=arn:aws:iam::NNNNNNNNNNN:role/AmazonEKS_EBS_CSI_DriverRole
```
# Install Pulsar
After completing the steps for OIDC ID and enabling Persistent in EKS, we can now install Pulsar using Helm.

```
mylaptop@DESKTOP:~$ kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.8.0/cert-manager.crds.yaml
mylaptop@DESKTOP:~$
mylaptop@DESKTOP:~$ helm install pulsar datastax-pulsar/pulsar --namespace pulsar --create-namespace -f dev-aws-auth-selfsigned-tls.yaml
```
# View EKS Pods, Services, and External IPs
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
```
# Cleanup and delete Pulsar
TODO - cleanup and removal of Persistent Volumes.  Command below does NOT remove PVs.  See issue with eksctl [https://github.com/kubernetes-sigs/aws-ebs-csi-driver/issues/1301](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/issues/1301)  
 
You must manually delete EBS volumes until this issue is fixed.  See EC2 Dashboard, Volumes sections for a list of PV and actions to remove/delete.  

```
mylaptop@DESKTOP:~$ eksctl delete cluster --name=mydemo --region=us-west-2 --wait
```
# Troubleshooting issues
For "pending volume" or "pending pod" conditions, see Troubleshooting guide at [Troubleshooting EKS Pod Pending issues:
https://aws.amazon.com/premiumsupport/knowledge-center/eks-troubleshoot-ebs-volume-mounts/
](Troubleshooting EKS Pod Pending issues:
https://aws.amazon.com/premiumsupport/knowledge-center/eks-troubleshoot-ebs-volume-mounts/
)  
  
Details on using and setup of EBS Persistent in EKS, see [And here:
https://docs.aws.amazon.com/eks/latest/userguide/managing-ebs-csi.html
](And here:
https://docs.aws.amazon.com/eks/latest/userguide/managing-ebs-csi.html
)  


