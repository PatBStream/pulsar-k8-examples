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
prometheus-pulsar-kube-prometheus-sta-prometheus-0     2/2     Running     0          72m
pulsar-adminconsole-7644d8ddfb-hhwx4                   1/1     Running     0          74m
pulsar-autorecovery-5c6b44c95-gmp6r                    1/1     Running     0          74m
pulsar-bastion-bb8c6dfd6-mpgdg                         1/1     Running     0          74m
pulsar-bookkeeper-0                                    1/1     Running     0          74m
pulsar-broker-0                                        1/1     Running     0          74m
pulsar-cert-manager-8cb7499bb-hxxgf                    1/1     Running     0          74m
pulsar-cert-manager-cainjector-75d587fdb9-qxrt7        1/1     Running     0          74m
pulsar-cert-manager-webhook-788994dcf6-kvchj           1/1     Running     0          74m
pulsar-function-0                                      2/2     Running     0          74m
pulsar-grafana-ff56446c5-xk2qm                         3/3     Running     0          74m
pulsar-kube-prometheus-sta-operator-6bc54f689d-hptbg   1/1     Running     0          74m
pulsar-kube-state-metrics-698f886fd5-tcgxh             1/1     Running     0          74m
pulsar-prometheus-node-exporter-tstq7                  1/1     Running     0          74m
pulsar-proxy-8f9c4d4db-hv9rq                           3/3     Running     0          74m
pulsar-pulsarheartbeat-857cfc8dc5-nbbth                1/1     Running     0          74m
pulsar-zookeeper-0                                     1/1     Running     0          74m
pulsar-zookeeper-metadata-jnh8x                        0/1     Completed   0          74m
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

## Access Pulsar-Admin and Pulsar-Shell
Pulsar-Admin and Pulsar-Shell access is available from the "pulsar-bastion-nnnn-nnn" pod.  To use this tools, start a interactive terminal session using kubectl.
```
mylaptop@DESKTOP:~$ kubectl exec -n pulsar -it $(kubectl get pods -n pulsar -l "component=bastion" -o jsonpath="{.items[0].metadata.name}") -- bash
```
# Access Pulsar Admin Console
Access Pulsar Admin Console from a localhost browser.

**IMPORTANT** Must run "minikube tunnel" before access

URL for Pulsar Admin Console:  https://localhost:443
# Access Grafana Dashboards
Access Grafana Dashboards from a localhost browser.

**IMPORTANT** Must run "minikube tunnel" before access
URL for Grafana:  http://localhost:3000

## Troubleshooting steps and workarounds
For troubleshooting steps and guidelines, see [RUN-README](RUN-README.md) section on **Troubleshooting steps and workarounds**.