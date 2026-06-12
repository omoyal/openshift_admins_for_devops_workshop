# Day 3 Lab 3 - Network Isolation & Cluster Storage

In this combined lab, we will secure our network perimeter using a **NetworkPolicy**, investigate the cluster's storage layer, and dynamically attach persistent **Storage** to our production environment.

---

## Part 1: The Network Isolation Challenge (Zero-Trust)

By default, all namespaces can talk to each other. Let's completely isolate the `prod-app-project` from the `dev-app-project`.

### 1. Deploy the Targets (As Admin 🛠️)
Log in as admin and spin up a backend deployment in Production and a testing pod in Development:
```bash
oc login [https://api.disco.lab:6443](https://api.disco.lab:6443) -u disco-admin -p Disco123!

# Deploy Prod Backend (As a Deployment) & Dev Attacker
oc create deployment prod-backend --image=quay.disco.lab/openshift/release/library/httpd:latest -n prod-app-project
oc run dev-attacker --image=quay.disco.lab/openshift/release/ubi8/ubi-minimal:latest -n dev-app-project -- sleep 3600
```

### 2. Test the Security Gap
Expose the production backend deployment and verify that the Dev pod can freely access it (This should succeed and return Apache HTML):
```bash
# Expose the deployment as a local service on port 80
oc expose deployment prod-backend --port=80 -n prod-app-project

# Test the connection from Dev to Prod
oc exec -it dev-attacker -n dev-app-project -- curl --connect-timeout 3 [http://prod-backend.prod-app-project.svc.cluster.local:80](http://prod-backend.prod-app-project.svc.cluster.local:80)
```

### 3. Apply the Firewall (NetworkPolicy)
Label the production namespace and apply an Ingress policy that blocks all incoming traffic unless it originates from within the production environment:
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
  podSelector: {} # Applies to all pods in this namespace
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          environment: production
EOF
```

### 4. Verify Isolation
Test the connection again. The request will now hang and **timeout**, proving our firewall successfully blocked the unprivileged Dev traffic:
```bash
oc exec -it dev-attacker -n dev-app-project -- curl --connect-timeout 3 [http://prod-backend.prod-app-project.svc.cluster.local:80](http://prod-backend.prod-app-project.svc.cluster.local:80)
```

---

## Part 2: Investigating & Allocating Cluster Storage

Now that the application is secure, let's look under the hood of the OpenShift storage layer and provision persistent disks.

### 5. Investigate the Storage Layer
List the available Storage Classes in the cluster to see which storage provisioners are ready to handle disk creation:
```bash
oc get storageclass
```
*(Identify the default provisioner marked with `(default)`).*

### 6. Create a PersistentVolumeClaim (PVC)
Request a 1Gi persistent disk from the cluster inside the production namespace:
```bash
cat << 'EOF' | oc apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prod-web-pvc
  namespace: prod-app-project
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF
```

### 7. Verify the Storage is Bound
Verify that OpenShift automatically generated a matching PersistentVolume (PV) and bound it to your request:
```bash
oc get pvc -n prod-app-project
```
*(The status should show **`Bound`**).*

---

## Part 3: Adding Storage to Our Application 💾

### 8. Mount the Disk to the Application
Attach the newly created PVC directly to our production web server deployment. We will mount it to the default Apache directory (`/var/www/html`):
```bash
oc set volume deployment/prod-backend --add --name=web-storage-volume --type=pvc --claim-name=prod-web-pvc --mount-path=/var/www/html -n prod-app-project
```

### 9. Verify the App is Running with the New Storage
Watch the deployment rollout a fresh pod containing our attached persistent disk:
```bash
oc get pods -n prod-app-project
```

**Boom! Network is isolated, storage is investigated, created, and successfully added to the application! 🚀**
