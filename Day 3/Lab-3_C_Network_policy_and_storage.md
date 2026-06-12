# Day 3 Lab 3 - Quick Network Isolation & Storage

Welcome to the final lab of Day 3!  
We are going to wrap up the day with a fast, high-impact exercise: closing a security gap between Dev and Prod, and attaching cluster storage to our application.

> [!IMPORTANT]
> **Our Mission:** Secure the network perimeter using a **NetworkPolicy** and attach persistent **Storage** to our application using *only* the `ubi-minimal` image we already mirrored.

---
<br><br><br>

---

> [!CAUTION]
> ### ⚠️ CRITICAL: Check Your Terminal Before You Enter!
> 
> To save yourself from a headache and avoid running commands in the wrong place, **always double-check your terminal prompt color and hostname** before hitting Enter.
> 
> | Terminal Color | Hostname | Network | What is it for? |
> | :--- | :--- | :--- | :--- |
> | 🟣 **Purple / Blue** | `lab-user@jump` | **"Lowside"** | **Connected to the Internet.** Use this host to download tools, pull container images, and prepare installation assets. |
> | 🟠 **Orange** | `lab-user@highside` | **"Highside"** | **Completely Air-Gapped (Disconnected).** This is where the actual lab happens. You will install the Quay mirror-registry and spin up your OpenShift cluster (`openshift.disco.lab`) here. |
> 
> #### 💡 Quick Tips:
> * **To switch** from the jump box to the disconnected environment, simply run: `ssh highside`
> * **Follow the colors!** The command boxes in your lab guide match these exact colors to show you exactly where to run them.

---

<br><br><br>





## Part 1: The 3-Minute Network Isolation Challenge

### 1. Launch the Deployments (As Admin 🛠️)
Make sure you are logged in as admin on the Highside and spin up both environments instantly using the official image path (the cluster will automatically mirror this to your local registry):

```bash
oc login https://api.disco.lab:6443 -u disco-admin -p Disco123!

# Create Production Deployment (Using the generic Red Hat path)
oc create deployment prod-backend --image=registry.redhat.io/ubi8/ubi-minimal:latest -n prod-app-project -- /bin/sh -c "sleep 3600"

# Expose the deployment as a Service so the internal DNS can resolve it!
oc expose deployment prod-backend --port=80 -n prod-app-project

# Create Development Attacker Pod
oc run dev-attacker --image=registry.redhat.io/ubi8/ubi-minimal:latest -n dev-app-project -- /bin/sh -c "sleep 3600"
```

<br><br>

### 2. Test the Security Gap
Let's try to reach Production from Dev. Since no web server is running, an **open network** will answer instantly with `Connection refused`:

```bash
oc exec -it dev-attacker -n dev-app-project -- curl --connect-timeout 3 http://prod-backend.prod-app-project.svc.cluster.local:80
```
> 🔌 **The Gap Confirmed:** The response returns instantly (`Connection refused`). This means the network allowed the traffic to pass right through!


<br>

### 3. Apply the Firewall (NetworkPolicy)
Let's label the production namespace and apply a strict firewall rule to drop all external incoming traffic:
```bash
# Label the namespace
oc label namespace prod-app-project environment=production

# Apply the NetworkPolicy
cat << 'EOF' | oc apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-dev-traffic
  namespace: prod-app-project
spec:
  podSelector: {} # Protects all pods in this namespace
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          environment: production # Only allow traffic from production namespaces
EOF
```

<br><br>


### 4. Verify the Firewall Works
Run the exact same command again. The request will now hang and **timeout**, proving the NetworkPolicy successfully intercepted and dropped the traffic:
```bash

oc exec -it dev-attacker -n dev-app-project -- curl --connect-timeout 3 http://prod-backend.prod-app-project.svc.cluster.local:80
```


---

<br><br><br>

## Part 2: Investigating & Adding Cluster Storage

### 5. Check Available Storage
See which StorageClass provider is ready to dynamically create disks for us:
```bash
oc get storageclass
```

### 6. Claim a Dynamic Disk (PVC)
Request a 1Gi persistent disk from the cluster's default storage provider:
```bash
cat << 'EOF' | oc apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prod-storage-pvc
  namespace: prod-app-project
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF
```

Check the status of your request:
```bash
oc get pvc -n prod-app-project
```

> [!NOTE]
> ** Don't Panic if it says `PENDING`!** > Modern clusters use a smart storage feature called `WaitForFirstConsumer`. This means OpenShift is intentionally freezing the disk creation until an actual Pod asks for it. It wants to see where the Pod is going to run first, so it can create the physical disk in the exact same rack/node!

<br>

### 7. Add the Storage to the Application
Attach and mount this new disk directly into our production deployment folder (`/mnt/data`):
```bash
oc set volume deployment/prod-backend --add --name=secure-data-vol --type=pvc --claim-name=prod-storage-pvc --mount-path=/mnt/data -n prod-app-project
```

### 8. Final Verification
Verify that the storage is successfully mounted inside the pod by checking the file system layout:
```bash
# Get the active pod name
oc get pods -n prod-app-project

# Run df -h inside the pod to see the mounted 1Gi disk
oc exec -it deployment/prod-backend -n prod-app-project -- df -h | grep mnt
```

**Congratulations! You isolated the network and attached persistent storage successfully! Day 3 Complete! 🚀🏆**
