- [Intro](#intro)
  - [Highlevel overview of setup](#highlevel-overview-of-setup)
  - [Assumptions](#assumptions)
  - [Install "eksctl" and "aws" Command-line tools](#install-eksctl-and-aws-command-line-tools)

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



