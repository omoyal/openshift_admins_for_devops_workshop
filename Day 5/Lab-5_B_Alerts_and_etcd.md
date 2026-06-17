# Lab 5.2: Maintance by Alerts & ETCD Backup-encryption

### Lab Objectives:
* Perform proactive infrastructure operations: Node Draining and configuration management.
* Manage Alerting rules and silences using the command line (`amtool`).
* Execute critical Day-2 recovery tasks: etcd backup and etcd encryption at rest.



---

## Part 1: Proactive SRE - Managing Node Pressure
Before a P1 occurs, you must know how to safely manipulate cluster infrastructure.

1. **Node Draining:** Pick a Worker Node. Safely evacuate all pods from it to prepare for maintenance.

```bash
oc get nodes
# take the name of one of Workers

oc adm drain <node-name> --ignore-daemonsets --delete-emptydir-data
# wait for complete draining
# TIP! You can see how openshift delete all pods from this node and moving them to another one)

# See the node NotScedhuled in cluster
oc get nodes
```


2. **Alert Simulation:** Open the `Alertmanager` UI or use `amtool` to check if any alerts were triggered by your drainage (hint: look for `KubeNodeNotReady` or `KubeNodeUnreachable`).

3. **Using amtool:** Use the CLI to silence the alert you just triggered for the next 15 minutes:

because we are in Air-gapped we dont have the amtool CLI, the CLI fond inside the AlertManager Pod

```bash
oc get pods -n openshift-monitoring
# take the name of one of alrtmanager pods
```

```bash
# 1. View active alerts (specifying the local URL)
oc exec -n openshift-monitoring alertmanager-main-0 -c alertmanager -- \
  amtool alert --alertmanager.url=http://localhost:9093
```

```bash
# 2. Silence the alert for 15 minutes
oc exec -n openshift-monitoring alertmanager-main-0 -c alertmanager -- \
  amtool silence add alertname="KubeNodeNotReady" \
  --alertmanager.url=http://localhost:9093 \
  --duration=15m --comment="Maintenance Window"
```



---

## Part 2: etcd Disaster Recovery
The etcd database is the cluster's heartbeat. We never touch it without a fresh backup.

1. **Backup:** Perform the backup. You can use `oc debug` method.

   **Using oc debug**
   ```bash
   # 1. Start a debug session on a master node
   oc debug node/<master-node-name>
   
   # 2. Switch to the host filesystem
   chroot /host
   
   # 3. Run the backup script
   sudo /usr/local/bin/cluster-backup.sh /home/core/assets/backup
   
   ```


2. **Verification:** Confirm the creation of the `snapshot.db` and the static pod resources file in the destination folder.

```bash
ls -lh /home/core/assets/backup

# exit from debug pod
Ctrl + D X 2
```

3. **Critical Thinking:** Why is it mandatory to run this script specifically on a Master node rather than from your local `oc` CLI?

💡 Answer: The cluster-backup.sh script relies on local filesystem access to the etcd data directory and the static pod manifests that define the etcd/API-server pods. The oc CLI communicates via the API, which may be unreachable if etcd is failing.

---

<br><br>

## Part 3: Protecting Data at Rest (etcd Encryption)

In a secure environment, etcd contents (including Secrets) must be encrypted.

1. **Status Check:** Check if encryption is currently enabled on the `KubeAPIServer` operator:
```bash
oc get kubeapiserver cluster -o=jsonpath='{.spec.encryption.type}'
# (If the output is empty, encryption is currently disabled)
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
# Watch the operators transition to 'Progressing'
watch oc get co kube-apiserver etcd

```



---

## Part 4: Cleanup

1. Uncordon the node you drained earlier to allow it to rejoin the scheduling pool:
```bash
oc adm uncordon <node-name>

```



```

```
