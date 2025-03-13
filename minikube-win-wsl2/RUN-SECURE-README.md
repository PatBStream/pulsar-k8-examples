- [After Pulsar Install](#after-pulsar-install)
  - [View the K8 Pods:](#view-the-k8-pods)
  - [Access Pulsar-Admin and Pulsar-Shell](#access-pulsar-admin-and-pulsar-shell)
- [Access Pulsar Admin Console](#access-pulsar-admin-console)
- [Access Grafana Dashboards](#access-grafana-dashboards)
  - [Troubleshooting steps and workarounds](#troubleshooting-steps-and-workarounds)

# After Pulsar Install

## View the K8 Pods:
Verify the K8 Pods are running with no errors.

```
mylaptop@DESKTOP:~$ kubectl get pods -n pulsar
NAME                                                   READY   STATUS      RESTARTS   AGE
prometheus-pulsar-kube-prometheus-sta-prometheus-0     2/2     Running     0          9m39s
pulsar-bookie-0                                        1/1     Running     0          9m54s
pulsar-bookie-init-vf6wg                               0/1     Completed   0          9m54s
pulsar-broker-0                                        1/1     Running     0          9m54s
pulsar-grafana-78f54fcd4d-xf7g4                        3/3     Running     0          9m54s
pulsar-kube-prometheus-sta-operator-6b69d65dfd-47qrb   1/1     Running     0          9m54s
pulsar-kube-state-metrics-67945fc86b-98g47             1/1     Running     0          9m54s
pulsar-prometheus-node-exporter-zpfz2                  1/1     Running     0          9m54s
pulsar-proxy-0                                         1/1     Running     0          9m54s
pulsar-pulsar-init-smwpq                               0/1     Completed   0          9m54s
pulsar-toolset-0                                       1/1     Running     0          9m54s
pulsar-zookeeper-0                                     1/1     Running     0          9m54s

mylaptop@DESKTOP:~$ kubectl get secret -n pulsar
NAME                                                            TYPE                 DATA   AGE
prometheus-pulsar-kube-prometheus-sta-prometheus                Opaque               1      11m
prometheus-pulsar-kube-prometheus-sta-prometheus-tls-assets-0   Opaque               1      11m
prometheus-pulsar-kube-prometheus-sta-prometheus-web-config     Opaque               1      11m
pulsar-ca-tls                                                   kubernetes.io/tls    3      11m
pulsar-grafana                                                  Opaque               3      11m
pulsar-kube-prometheus-sta-admission                            Opaque               3      11m
pulsar-mini-token-admin                                         Opaque               2      16m
pulsar-mini-token-asymmetric-key                                Opaque               2      17m
pulsar-mini-token-broker-admin                                  Opaque               2      17m
pulsar-mini-token-proxy-admin                                   Opaque               2      17m
pulsar-pulsar-manager-secret                                    Opaque               4      11m
pulsar-tls-bookie                                               kubernetes.io/tls    3      11m
pulsar-tls-broker                                               kubernetes.io/tls    3      11m
pulsar-tls-proxy                                                kubernetes.io/tls    3      11m
pulsar-tls-recovery                                             kubernetes.io/tls    3      11m
pulsar-tls-toolset                                              kubernetes.io/tls    3      11m
pulsar-tls-zookeeper                                            kubernetes.io/tls    3      11m
pulsar-token-admin                                              Opaque               2      16m
pulsar-token-asymmetric-key                                     Opaque               2      16m
pulsar-token-broker-admin                                       Opaque               2      16m
pulsar-token-proxy-admin                                        Opaque               2      16m
sh.helm.release.v1.pulsar.v1                                    helm.sh/release.v1   1      11m

```
**NOTE** - If TLS is enabled correctly, you will see "pulsar-tls..." in the list of secrets from K8.

# Test Pulsar TLS with Openssl client
Test TLS connections using Openssl client.

## Connect to Pulsar Toolset Pod
Connect to the ToolSet Pod and run the Openssl Client command below.  Look for the "CONNECTED" message to ensure TLS is enabled and working.
```
mylaptop@DESKTOP:~$ kubectl exec -it -n pulsar pulsar-toolset-0 -- /bin/bash

pulsar-toolset-0:/pulsar$ openssl s_client -connect pulsar-broker:6651 -showcerts -CAfile /pulsar/certs/ca/ca.crt

Connecting to 10.244.0.18
CONNECTED(00000003)
Can't use SSL_get_servername
depth=1 CN=pulsar.svc.cluster.local
verify return:1
depth=0 O=pulsar, CN=pulsar-broker
verify return:1
---
Certificate chain
 0 s:O=pulsar, CN=pulsar-broker
   i:CN=pulsar.svc.cluster.local
   a:PKEY: rsaEncryption, 4096 (bit); sigalg: RSA-SHA256
   v:NotBefore: Mar 13 19:20:43 2025 GMT; NotAfter: Jun 11 19:20:43 2025 GMT
-----BEGIN CERTIFICATE-----
MIIEbTCCA1WgAwIBAgIRALySv/+jndwqUXDb+keGZVcwDQYJKoZIhvcNAQELBQAw
IzEhMB8GA1UEAxMYcHVsc2FyLnN2Yy5jbHVzdGVyLmxvY2FsMB4XDTI1MDMxMzE5

```
**NOTE** The connection will timeout due as the expected Pulsar Client data exchange does not offer.  This only test the TLS handshake and connectivity.

After all Pods are running and/or completed, run minikube service command to access Pulsar Manager, Pulsar Proxy, and Grafana/Prometheus Dashboards.

```
mylaptop@DESKTOP:~$ minikube service pulsar-grafana -n pulsar

```
## Note About Pulsar Manager and TLS
If your Pulsar Cluster has TLS enabled, additional setup of Pulsar Manager is needed.  Refer to [Pulsar Manager](https://github.com/apache/pulsar-manager/blob/master/README.md) docs for details.
## Access Pulsar-Admin and Pulsar-Shell
Pulsar-Admin and Pulsar-Shell access is available from the "pulsar-toolset-0" pod.  To use this tools, start a interactive terminal session using kubectl.
```
mylaptop@DESKTOP:~$ kubectl exec -it -n pulsar pulsar-toolset-0 -- /bin/bash

```
# Access Grafana Dashboards
Access Grafana Dashboards from a localhost browser.

**IMPORTANT** Must run "minikube service" before access, minikube service pulsar-grafana -n pulsar

## Troubleshooting steps and workarounds
For troubleshooting steps and guidelines, see [RUN-README](RUN-README.md) section on **Troubleshooting steps and workarounds**.