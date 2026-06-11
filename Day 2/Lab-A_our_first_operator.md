# Day 2 Lab 1 - Our First Operator

<br>

In this lab, we will simulate a very common real-world scenario: bringing an external operator into our environment and installing it inside the cluster. We will do this together, step-by-step.

> [!IMPORTANT]
> Please make sure to read and understand the commands before running them, so you know exactly what they do.
> If you have any questions at any point, don't hesitate to ask the instructor.
> 
> Best of luck! 💪

---


<br><br><br>

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




## Part 1: Downloading the Operator (On the Internet Side)
> [!CAUTION]
> **SERVER CHECK:** Perform the following steps ONLY on the **Jump Server** (🟣 Purple/Blue prompt).

<br>

### 1. Create the ImageSetConfiguration
We will create a dedicated directory under `/mnt/low-side-data` and write our configuration file there.

```bash
cd /mnt/low-side-data/
mkdir gitops-operator
cd /gitops-operator
```

Create and Paste the following content into `imageset-config.yaml`:
```bash
vim imageset-config.yaml
```

```yaml
---
kind: ImageSetConfiguration
apiVersion: mirror.openshift.io/v2alpha1
mirror:
  operators:
    - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.17
      packages:
        - name: openshift-gitops-operator
          channels:
            - name: latest
```
<br>

### 2. Download via oc-mirror
Run the mirror command to pull the operator assets from the internet:
```bash
oc mirror -c imageset-config.yaml file:///mnt/low-side-data/gitops-operator --v2
```

> [!TIP]
> Wait until the process finishes and you see the following message:
> `" 👋 Goodbye, thank you for using oc-mirror "`

<br>

### 3. Transfer the Assets to the Air-Gapped Environment

Now, move the generated TAR output over to our disconnected environment using `rsync`:
```bash
rsync -avP /mnt/low-side-data/gitops-operator highside:/mnt/high-side-data/
```
*Wait until the file copy is completely done.*

---
<br><br><br>
## Part 2: Installing the Operator (In the Air-Gapped Env)
> [!WARNING]
> **SERVER CHECK:** We are now moving into the Air-Gapped environment. Switch to the **Highside Server** (🟠 Orange prompt).


<br>

### 4. Connect and Verify
SSH into the Highside server and navigate to our copied data directory:
```bash
ssh highside
cd /mnt/high-side-data/gitops-operator
```

<br>

### 5. Push the Operator to the Local Registry
Push the mirrored assets into our local Quay registry (make sure you are logged in if needed):
```bash
oc mirror -c <image_set_configuration> --from file:///mnt/high-side-data/gitops-operator docker://$(hostname):8443 --v2
```
*Wait for oc-mirror to complete successfully.*

<br>


### 6. Investigate the Generated Manifests
Let's see what files were created as a result of the mirroring action:
```bash
ls -lah
ls -lah working-dir/cluster-resources/
```
*(Take a moment to read and inspect the generated files)*

---
<br><br><br>


## Part 3: Cluster Configuration & Deployment
<br>

### 7. Disable Default CatalogSources
Before we apply our new files to OpenShift, we must disable the default online catalogs.

> [!NOTE]
> **The Logic:** Since OpenShift doesn't know whether it has internet access or not, it will constantly throw errors trying to pull the unavailable indexes from the internet. Therefore, we disable the default ones to keep our cluster healthy and clean.

<br>


**7.a. Log in to the cluster from the CLI:**
```bash
oc login -u kubeadmin [https://api.disco.lab:6443](https://api.disco.lab:6443)
```
*(Note: You can find your kubeadmin password on the highside server at: `/mnt/high-side-data/auth/kubeadmin-password`)*

<br>


**7.b. Apply the patch to disable default sources:**
```bash
oc patch OperatorHub cluster --type merge -p '{"spec": {"disableAllDefaultSources": true}}'
```

<br>


### 8. Deploy the Mirrored Kubernetes Objects
Now, apply the generated manifests to register our local operator catalog in the cluster:
```bash
oc apply -f /mnt/high-side-data/gitops/working-dir/cluster-resources/
```

---
<br><br><br>
## Part 4: Verification

<br>

### 9. Monitor the Deployment Status
Run the following commands in your terminal to verify everything is climbing up correctly:
```bash
oc get mcp
oc get pods -n openshift-marketplace
```
### 📋 What just happened behind the scenes?
* Because the **IDMS** (`ImageDigestMirrorSet`) needs to update each node in the cluster, the **MCP** (*MachineConfigPool*) will begin updating.
* Inside the `openshift-marketplace` namespace, you will see a new pod appearing – this pod runs our newly mirrored CatalogSource index.
* Open your **OpenShift Web Console**, navigate to **Operators** -> **OperatorHub**, and look for your new operator!

---

# Congratulations! Well Deserved! 🎉
