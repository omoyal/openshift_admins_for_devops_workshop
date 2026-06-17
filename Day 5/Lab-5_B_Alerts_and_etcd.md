# Lab 5.2: Infrastructure Maintenance & Disaster Recovery

### Lab Objectives:
* Perform proactive infrastructure operations: Node Draining and configuration management.
* Manage Alerting rules and silences using the command line (`amtool`).
* Execute critical Day-2 recovery tasks: etcd backup and etcd encryption at rest.



---

## Part 1: Proactive SRE - Managing Node Pressure
Before a P1 occurs, you must know how to safely manipulate cluster infrastructure.

1. **Node Draining:** Pick a Worker Node. Safely evacuate all pods from it to prepare for maintenance.
   ```bash
   oc adm drain <node-name> --ignore-daemonsets --delete-emptydir-data

```

2. **Alert Simulation:** Open the `Alertmanager` UI or use `amtool` to check if any alerts were triggered by your drainage (hint: look for `KubeNodeNotReady` or `KubeNodeUnreachable`).
3. **Using amtool:** Use the CLI to silence the alert you just triggered for the next 15 minutes:
```bash
# Find the Alert ID first
amtool alert
# Silence the alert
amtool silence add alertname="KubeNodeNotReady" --duration=15m --comment="Maintenance Window"

```



---

## Part 2: etcd Disaster Recovery

The etcd database is the cluster's heartbeat. We never touch it without a fresh backup.

1. **Backup:** Connect to a Master node via SSH and execute the built-in backup script.
```bash
# Run on a master node
sudo /usr/local/bin/cluster-backup.sh /home/core/assets/backup

```


2. **Verification:** Confirm the creation of the `snapshot.db` and the static pod resources file in the destination folder.
3. **Critical Thinking:** Why is it mandatory to run this script specifically on a Master node rather than from your local `oc` CLI?

---

## Part 3: Protecting Data at Rest (etcd Encryption)

In a secure environment, etcd contents (including Secrets) must be encrypted.

1. **Status Check:** Check if encryption is currently enabled on the `KubeAPIServer` operator:
```bash
oc get kubeapiserver cluster -o=jsonpath='{.spec.encryption.type}'

```


2. **Apply Encryption:** Update the `KubeAPIServer` configuration to enable AES-CBC encryption:
```bash
oc edit kubeapiserver cluster

```


*Modify the spec to look like this:*
```yaml
spec:
  encryption:
    type: aescbc

```


3. **Monitoring the Rollout:** The cluster will now automatically re-encrypt every Secret and ConfigMap. Monitor the progress of the `KubeAPIServer` and `etcd` operators:
```bash
oc get co kube-apiserver
oc get co etcd

```



---

## Part 4: Cleanup

1. Uncordon the node you drained earlier to allow it to rejoin the scheduling pool:
```bash
oc adm uncordon <node-name>

```



```

```
