- [After Pulsar Install](#after-pulsar-install)
  - [View the K8 Pods:](#view-the-k8-pods)
  - [Access Pulsar-Admin and Pulsar-Shell](#access-pulsar-admin-and-pulsar-shell)
  - [Enable "JMS" support in Pulsar-Admin and Pulsar-Shell](#enable-jms-support-in-pulsar-admin-and-pulsar-shell)
  - [Troubleshooting steps and workarounds](#troubleshooting-steps-and-workarounds)
- [Access Pulsar Admin Console](#access-pulsar-admin-console)
- [Access Grafana Dashboards](#access-grafana-dashboards)
- [Example Helm Value Override with Pulsar HeartBeat](#example-helm-value-override-with-pulsar-heartbeat)

# After Pulsar Install

## View the K8 Pods:
Verify the K8 Pods are running with no errors.

```
mylaptop@DESKTOP:~$ kubectl get pods -n pulsar
NAME                                                  READY   STATUS      RESTARTS   AGE
prometheus-pulsar-kube-prometheus-sta-prometheus-0    2/2     Running     0          3m58s
pulsar-bookie-0                                       1/1     Running     0          4m19s
pulsar-bookie-init-cg7jv                              0/1     Completed   0          4m19s
pulsar-broker-0                                       1/1     Running     0          4m19s
pulsar-grafana-76c4767bb5-prf4c                       3/3     Running     0          4m19s
pulsar-kube-prometheus-sta-operator-d9db77ffc-gp5zt   1/1     Running     0          4m19s
pulsar-kube-state-metrics-64c5f85bd7-tk4jh            1/1     Running     0          4m19s
pulsar-prometheus-node-exporter-vs6t9                 1/1     Running     0          4m19s
pulsar-proxy-0                                        1/1     Running     0          4m19s
pulsar-pulsar-init-g28m6                              0/1     Completed   0          4m19s
pulsar-pulsar-manager-0                               1/1     Running     0          4m19s
pulsar-pulsar-manager-init-flppl                      0/1     Completed   0          4m19s
pulsar-toolset-0                                      1/1     Running     0          4m19s
pulsar-zookeeper-0                                    1/1     Running     0          4m19s
```
After all Pods are running and/or completed, run minikube service commands to access Pulsar Manager, Pulsar Proxy, and Grafana/Prometheus Dashboards.

```
mylaptop@DESKTOP:~$ minikube service -n pulsar pulsar-pulsar-manager
|-----------|-----------------------|-------------|---------------------------|
| NAMESPACE |         NAME          | TARGET PORT |            URL            |
|-----------|-----------------------|-------------|---------------------------|
| pulsar    | pulsar-pulsar-manager | server/9527 | http://192.168.49.2:32562 |
|-----------|-----------------------|-------------|---------------------------|
🏃  Starting tunnel for service pulsar-pulsar-manager.
|-----------|-----------------------|-------------|------------------------|
| NAMESPACE |         NAME          | TARGET PORT |          URL           |
|-----------|-----------------------|-------------|------------------------|
| pulsar    | pulsar-pulsar-manager |             | http://127.0.0.1:40629 |
|-----------|-----------------------|-------------|------------------------|
🎉  Opening service pulsar/pulsar-pulsar-manager in default browser...
👉  http://127.0.0.1:40629
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.

```
Now, run these commands in separate terminals for access to Proxy and Grafana Dashboards.
```
mylaptop@DESKTOP:~$ minikube service -n pulsar pulsar-grafana

mylaptop@DESKTOP:~$ minikube service -n pulsar pulsar-proxy

```
## How to setup and find Pulsar Manager ID and Password
Initial ID is "pulsar" and the password for Pulsar Manager can be found using the K8 commands below:
```
mylaptop@DESKTOP:~$ export UI_PASSWORD=$(kubectl get secret pulsar-pulsar-manager-secret -n pulsar -o
jsonpath="{.data.UI_PASSWORD}")

mylaptop@DESKTOP:~$ echo $UI_PASSWORD | base64 --decode
Zud0HVBeRped6Kge2PoPmV8pnxypzbLc

```
## Access Pulsar-Admin and Pulsar-Shell
Pulsar-Admin and Pulsar-Shell access is available from the "pulsar-toolset-0" pod.  To use this tools, start a interactive terminal session using kubectl.
```
mylaptop@DESKTOP:~$ kubectl exec -it -n pulsar pulsar-toolset-0 -- /bin/bash
pulsar$ bin/pulsar-admin
```
## Enable "JMS" support in Pulsar-Admin and Pulsar-Shell
Once in the session from the pulsar-bastion pod, goto **conf** directory to manually edit the **client.conf** file to enable JMS support in these tools.

NOTE:  To enable "jms" support in pulsar-admin and pulsar-shell, update file conf/client.conf parameters:
```
#Pulsar Admin Custom Commands
customCommandFactoriesDirectory=commandFactories
customCommandFactories=jms
```

## Troubleshooting steps and workarounds

If errors occurs and/or pods never goto a **Running** or **Completed** state, you may see:
```
mylaptop@DESKTOP:~$ kubectl get pods -n pulsar
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
**IMPORTANT:** If this error occurs, try **pre downloading** the **images** into Minikube.
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
# Access Pulsar Manager
Access Pulsar Manager from a localhost browser.

**IMPORTANT** Must run "minikube service" before access
```
mylaptop@DESKTOP:~$ minikube service -n pulsar pulsar-pulsar-manager
```

# Access Grafana Dashboards
Access Grafana Dashboards from a localhost browser.

**IMPORTANT** Must run "minikube service" before access
URL for Grafana:  http://localhost:3000

# Example Helm Value Override with Pulsar HeartBeat
You can use a **value** file to override default values in Pulsar Helm chart, to configure specific options.  For example, configure Pulsar HeartBeat to use a specific Pulsar topic and message size settings.

An example of this is in file [dev-heartbeat-config.yaml](helm-values/dev-heartbeat-config.yaml)
```
pulsarHeartbeat:
  component: pulsarheartbeat
  config:
    latencyTest:
      intervalSeconds: 10
      topicName: "persistent://public/default/pubsub-lat-test"
      
```