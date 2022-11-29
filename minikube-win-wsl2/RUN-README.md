- [After Pulsar Install](#after-pulsar-install)
  - [View the K8 Pods:](#view-the-k8-pods)
  - [Troubleshooting steps and workarounds](#troubleshooting-steps-and-workarounds)

# After Pulsar Install

## View the K8 Pods:
Verify the K8 Pods are running with no errors.

```
mylaptop@DESKTOP:~$ kubectl get pods
NAME                                                   READY   STATUS      RESTARTS   AGE
prometheus-pulsar-kube-prometheus-sta-prometheus-0     2/2     Running     2          98m
pulsar-adminconsole-685ff487cf-nv6zh                   1/1     Running     1          99m
pulsar-autorecovery-7599455c4-hjn28                    1/1     Running     1          99m
pulsar-bastion-76b9bfcbb8-x7bxd                        1/1     Running     1          99m
pulsar-bookkeeper-0                                    1/1     Running     1          99m
pulsar-broker-d9787655c-m4p45                          1/1     Running     1          99m
pulsar-function-0                                      2/2     Running     4          99m
pulsar-grafana-66c5c84b6c-6g5vc                        3/3     Running     3          99m
pulsar-kube-prometheus-sta-operator-6bc54f689d-8djcb   1/1     Running     1          99m
pulsar-kube-state-metrics-698f886fd5-2jmlr             1/1     Running     2          99m
pulsar-prometheus-node-exporter-t8dq6                  1/1     Running     1          99m
pulsar-proxy-6c866c9dc5-vcxbn                          3/3     Running     3          99m
pulsar-pulsarheartbeat-785f697fcb-94sq8                1/1     Running     1          99m
pulsar-zookeeper-0                                     1/1     Running     1          99m
pulsar-zookeeper-metadata-dwthl                        0/1     Completed   0          99m
```
After all Pods are running and/or completed, run minikube tunnel command to access Pulsar Admin, Pulsar Proxy, and Grafana/Prometheus Dashboards.

```
mylaptop@DESKTOP:~$ minikube tunnel
✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

❗  The service/ingress pulsar-adminconsole requires privileged ports to be exposed: [80 443]
🔑  sudo permission will be asked for it.
🏃  Starting tunnel for service pulsar-adminconsole.
🏃  Starting tunnel for service pulsar-grafana.
🏃  Starting tunnel for service pulsar-proxy.

```


## Troubleshooting steps and workarounds

If errors occurs and/or pods never goto a **Running** or **Completed** state, you may see:
```
mylaptop@DESKTOP:~$ kubectl get pods
NAME                                                   READY   STATUS                  RESTARTS   AGE
prometheus-pulsar-kube-prometheus-sta-prometheus-0     0/2     PodInitializing         0          13m
pulsar-bookie-0                                        0/1     Init:0/1                0          13m
pulsar-bookie-init-6xmbl                               0/1     Init:ImagePullBackOff   0          13m
pulsar-broker-0                                        0/1     Init:ImagePullBackOff   0          13m
```
If this occurs during the Pod startup, look for errors using the kubectl describe command

This example, **describes** the pod "pulsar-mini-toolset-0" and shows the erros:  **Failed to pull image "apachepulsar/pulsar-all:2.10.2": rpc error: code = Unknown desc = context deadline exceeded**
```
mylaptop@DESKTOP:~$ kubectl describe pod pulsar-mini-toolset-0 -n pulsar
.
.
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  20m                 default-scheduler  Successfully assigned pulsar/pulsar-mini-pulsar-init-nnjhd to minikube
  Normal   BackOff    18m                 kubelet            Back-off pulling image "apachepulsar/pulsar-all:2.10.2"
  Warning  Failed     18m                 kubelet            Error: ImagePullBackOff
  Warning  Failed     2m9s (x2 over 18m)  kubelet            Failed to pull image "apachepulsar/pulsar-all:2.10.2": rpc error: code = Unknown desc = context deadline exceeded
```
If this error occurs, try **pre downloading** the **images** into Minikube.
```
mylaptop@DESKTOP:~$ minikube image pull apachepulsar/pulsar-all:2.10.2 
```
Verify images in minikube with:
```
mylaptop@DESKTOP:~$ minikube image ls
quay.io/prometheus/prometheus:v2.35.0
quay.io/prometheus/node-exporter:v1.3.1
quay.io/prometheus-operator/prometheus-operator:v0.56.3
quay.io/prometheus-operator/prometheus-config-reloader:v0.56.3
quay.io/kiwigrid/k8s-sidecar:1.15.6
k8s.gcr.io/pause:3.4.1
.
.
.
```