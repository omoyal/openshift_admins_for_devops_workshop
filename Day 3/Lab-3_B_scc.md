# Day 3 Lab 2 - The SCC Challenge (Handling Legacy Applications)

Now that our corporate access boundaries are set, our developer (`disco-dev`) wants to deploy a legacy financial microservice inside the `dev-app-project`. 

However, OpenShift enforces strict operating system-level security constraints right out of the box (`restricted-v2`). Let's see what happens when legacy code meets enterprise security!

> [!IMPORTANT]
> **Our Mission:** First, mirror the required base image into our air-gapped environment. Then, experience a real-world deployment failure, diagnose it using SRE troubleshooting commands, and fix it using a dedicated `ServiceAccount` and **SCC** policies.

---

## Part 0: SRE Pre-requisite - Mirroring the Image (oc-mirror v2)

Since our Highside environment is completely air-gapped, we must first pull the required container image from the internet using our Jump Server and push it into our local Quay registry.

### 1. Download the Asset (On Jump Server 🟣)
Create an image set configuration file on your **Jump Server** to target the required image:

```bash
cat << 'EOF' > /tmp/imageset-config.yaml
apiVersion: mirror.openshift.io/v1alpha2
kind: ImageSetConfiguration
mirror:
  additionalImages:
  - name: registry.redhat.io/ubi8/ubi-minimal:latest
EOF
```

Run `oc-mirror` to download the image to your local disk, archive it, and transfer it over the network to the Highside server:

```bash
# Download the image
oc-mirror --config=/tmp/imageset-config.yaml file://mirror-disk

# Archive the data
tar -czf /tmp/mirror-data.tar.gz mirror-disk/

# Transfer to Highside
scp /tmp/mirror-data.tar.gz highside:/tmp/
```

### 2. Publish to Local Registry (On Highside Server 🟠)
Switch to your **Highside Server**, extract the data, and push the image directly into your local enterprise Quay registry:

```bash
# SSH into Highside
ssh highside

# Extract the archive
cd /tmp
tar -xzf /tmp/mirror-data.tar.gz

# Push to local Quay
oc-mirror --from file://mirror-disk docker://quay.disco.lab/openshift/release
```

---

## Part 1: The Crash (Executed as Developer 🧑‍💻)

### 3. Log in as the Developer
Now that the image is safe inside our network, let's log in as the restricted developer account:

```bash
oc login [https://api.disco.lab:6443](https://api.disco.lab:6443) -u disco-dev -p Disco123!
```

### 4. Deploy the Legacy Application
Attempt to deploy the legacy banking application, pointing to our newly mirrored local image. Notice that it explicitly demands to run as the `root` user (`runAsUser: 0`):

```bash
cat << 'EOF' | oc apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: legacy-banking-app
  namespace: dev-app-project
spec:
  replicas: 1
  selector:
    matchLabels:
      app: legacy-vault
  template:
    metadata:
      labels:
        app: legacy-vault
    spec:
      containers:
      - name: banking-container
        image: quay.disco.lab/openshift/release/ubi8/ubi-minimal:latest
        command: ["/bin/sh", "-c", "echo 'Banking system running as root!' && sleep 3600"]
        securityContext:
          runAsUser: 0
EOF
```

---

## Part 2: SRE Troubleshooting 🕵️

### 5. Check the Pod Status
Let's see if our application managed to clear the launchpad:

```bash
oc get pods -n dev-app-project
```
*Notice anything? The Pod isn't even being created, or it is stuck completely!*

### 6. Inspect the Deployment Warnings
Since there is no pod to inspect, we must look at the **Events** layer to see why the cluster is refusing to build the pod:

```bash
oc get events -n dev-app-project --sort-by='.metadata.creationTimestamp'
```

> [!CAUTION]
> **The SRE Diagnosis:** You will spot a critical warning error explicitly stating that the pod violates the **`restricted-v2`** Security Context Constraint (SCC) because it is trying to run with UID 0 (root). OpenShift blocked it at the admission webhook level!

---

## Part 3: The SRE Fix (Executed as Admin 🛠️)

To fix this without globally compromising the cluster's safety, we will isolate this application by running it under a custom **ServiceAccount** and granting *only* that specific account the permission to bypass the root restriction.

### 7. Switch to the Admin Account
Only cluster administrators can manipulate system-level security constraints (SCCs):

```bash
oc login [https://api.disco.lab:6443](https://api.disco.lab:6443) -u disco-admin -p Disco123!
```

### 8. Create a Dedicated ServiceAccount
Create a distinct system identity for our legacy app inside the dev namespace:

```bash
oc create serviceaccount legacy-sa -n dev-app-project
```

### 9. Grant the "anyuid" Security Constraint
We will attach the built-in **`anyuid`** SCC role to our new ServiceAccount. This tells OpenShift: *"Allow whoever uses this specific account to run with any User ID they want, including root."*

```bash
oc adm policy add-scc-to-user anyuid -z legacy-sa -n dev-app-project
```
> [!NOTE]
> **oc adm policy add-scc-to-user** (assigns an OS-level Security Context Constraint role) **anyuid** (the specific SCC policy that permits any container UID, including root) **-z** (shorthand flag pointing directly to a ServiceAccount name) **legacy-sa** (the target service account identity).

---

## Part 4: Testing the Resolution 🧪

### 10. Patch the Application to Use the ServiceAccount
Now, let's update our deployment to assume this new secure identity:

```bash
oc set serviceaccount deployment/legacy-banking-app legacy-sa -n dev-app-project
```

### 11. Verify the Pod is Live
Watch OpenShift pick up the change, apply the `anyuid` rules, and successfully deploy the container:

```bash
oc get pods -n dev-app-project -w
```
*(Press `Ctrl+C` to exit the watch mode once the pod shows `Running`)*

**Boom! You just mirrored the image, diagnosed the security block, and solved a classic OpenShift SCC roadblock like a pro SRE! 🚀**
