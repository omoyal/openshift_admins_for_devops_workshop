# Day 2 Lab 2 - Configure our Operators

<br>

In this lab, we will take the next step in our journey. Now that we have successfully mirrored the operator using `oc-mirror`, pushed it to our local registry, applied the required objects to the cluster, and verified its presence in the **OperatorHub**, it's time to dive deeper! 

We will explore the underlying components we discussed in class, complete the actual installation of the operator, and maybe even spin up a live instance. 

> [!IMPORTANT]
> Just like the previous labs, we will walk through this process together, step-by-step. 
> Take your time to understand each step!

**Good luck!** 🚀

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



## Part 1: Operator Investigation

> [!WARNING]
> **SERVER CHECK:** We are now working in the Air-Gapped environment. Switch to the **Highside Server** (🟠 Orange prompt).

<br>

### 1. Verify Cluster Connection
First, verify that you are successfully logged in to the cluster from the CLI.


<br><br>


### 2. Investigate OLM Components via CLI
Let's use the CLI to investigate the core Operator Lifecycle Manager (OLM) components: **Subscription**, **CSV**, **OperatorGroup**, and **InstallPlan**.

**2.a. Subscription**
Run the following command to locate our subscription:
```bash
$ oc get subscription -A
```
> [!TIP]
> The flag `-A` means "All Namespaces" – it will search everywhere across the cluster for you.

<img width="1317" height="45" alt="image" src="https://github.com/user-attachments/assets/a46b8cf3-4db1-4176-989b-3e2879bd4401" />

You should see our operator's Subscription in the output table showing its Namespace (NS), Source, Channel, etc.

Now, let's inspect what this subscription contains in detail:
```bash
$ oc get subscription openshift-gitops-operator -n openshift-gitops-operator -o yaml
```
*(Explanation: **oc** calls the API, **get** specifies the action, **openshift-gitops-operator** is the object name, **-n** targets the namespace it resides in, and **-o yaml** outputs the resource in clean YAML format).*

Take a look into the generated YAML structure.

**2.b. CSV (Cluster Service Version)**
Run a similar command to view the ClusterServiceVersion (CSV) objects:
```bash
$ oc get csv -A
```

**2.c. OperatorGroup**
Run a similar command to see the OperatorGroup objects:
```bash
$ oc get og -A
```

**2.d. InstallPlan**
At this moment, you won't see an active InstallPlan for this deployment because we haven't triggered the actual installation yet.

---

> [!TIP]
> **Prefer the Web Console?** You can also investigate all of these objects through the UI:
> 1. Log in to the OpenShift Web Console.
> 2. In the left-side menu, navigate to **Home** -> **Search**.
> 3. Set **Project: All Projects**.
> 4. Under **Resources**, select your object type (e.g., `Subscription`).
> 5. Click on the resource and explore the **Details** and **YAML** tabs.

<br><br>

---

## Part 2: Operator Installation

Now that you've seen everything related to the operator and have absolutely mastered it, let's go ahead and install the operator! 🚀

### 3. Trigger Installation via Web Console
1. Open the OpenShift Web Console (log in if prompted).
2. In the left-side menu, navigate to **Operators** -> **OperatorHub**.
3. Search for and select our Operator, then click **Install**.
4. On the installation page, review the configuration details. You don't need to change anything.
   *(You will see options we discussed in class, such as the Update Strategy, Target Namespace, etc.)*
5. Click **Create / Install**.

### 4. Monitor the Background Process
Behind the scenes, the magic is happening. Let's trace it!

**4.a. Monitor the Installation Pod & Job**
Duplicate your current browser tab. In the new tab, navigate to **Workloads** -> **Pods** and select the project **`openshift-marketplace`**. 
You will see a new pod running that manages the installation process (feel free to check its logs!). This pod was created by a Job (you can verify this by checking **Workloads** -> **Jobs**).

**4.b. Inspect the InstallPlan**
While the installation is running, navigate to **Home** -> **Search**, select **All Projects**, and choose **InstallPlan** as the resource. You will see our active InstallPlan. Click on it to investigate its **YAML**, **Details**, and **Components** tabs.

**4.c. Confirm Successful Installation**
Wait for the operator to install successfully (this may take a couple of minutes). Once finished, you will be able to view it under **Operators** -> **Installed Operators**.

<br><br>

---

# Congratulations! 🎉


























<br><br><br>
<br><br><br>
<br><br><br>
<br><br><br>
<br><br><br>
<br><br><br>
<br><br><br>
<br><br><br>
<br><br><br>
<br><br><br>
<br><br><br>
<br><br><br>
<br><br><br>
<br><br><br>
<br><br><br>
<br><br><br>


## Part 1: Operator Investing
> [!WARNING]
> **SERVER CHECK:** We are now working into the Air-Gapped environment. Switch to the **Highside Server** (🟠 Orange prompt).

<br>




1. check you are login to the cluster from CLI
2. in the CLI lets investing about the component: Subscription, Csv, OperatorGroup and InstallPlan
   2.a. **subscription**
   $ oc get subscription -A
   (Tip The fleg -A mean search for me in all around the cluster)
screenshot:
<img width="1317" height="45" alt="image" src="https://github.com/user-attachments/assets/a46b8cf3-4db1-4176-989b-3e2879bd4401" />
   you need to see our operator Subscription under the table (NS, Source,Channel etc..)

   lets see what this sub contain:
   $ oc get subscription openshift-gitops-operator -n openshift-gitops-operator -o yaml
   (explain: **oc** (call the API) **get** (the action' like create delete...) **<the name of object>** **-n** (the namespace is contain) **-o yaml** (give me that in yaml for clearly))

   look into the Yaml
   
   2.b. **CSV (Cluster Service Version)**
   replace same command but to see the csv object
   $ oc get csv -A
   ...


   2.c. **OperatorGroup (Cluster Service Version)**
   replace same command but to see the OperatorGroup object
   $ oc get og -A
   ...

   2.d **Install Plan**
   in this moment we dont see anything because we not install the operator


   TIP: If you want you can go to the Web-console and investing the all object there:
   Login to Disco-web-Openshift-Clister
   in the Menu(left-side) Under the HOME -> Search -> Project: All Projects -> Resources -> choose one: like subscription for example -> investing the "Detailes" and "Yaml" Tabs




## Part 2: Operator Installation
Now that you've seen everything related to the operator and have absolutly mastered it, let's go ahead and install the operator! 🚀
3. go to the Web-openshift-console (Login if need)
in the Menu(left-side), choose 'Operators' -> 'OperatorHub' -> choose our Operator -> choose Install
in this page brief the dtailes, Dont need to change anything
( You wiil see the Object the option that ask them in the class, like: Updated starategy, Ns, etc...)
-> choose create

4. in this moment the magic happen the background, lets see.
4.a. Diplicate the Tab -> in the new tab go to Menu -> Workload -> Pods -> choose Project 'Openshift-marketPlace' -> you can see a new pod runing - this pod manage the installation (Check it logs) -> the Pod create from a Job (if you go to Workloads -> Jobs you can see the job).

4.b.
Now after the Operator Installation started Go to Menu -> Search -> All Projects -> Resource: InstallPlan  - you cans see our installPlan, investing it (Click it and see the Yaml, Deatlies and Components)

4.c.
Wait the Operator Installed susccssefuly, this is take while minutes.
when the operator finish the installation you can see it under -> operators -> installed Operators


COngretulations!!!!!













### 1. Create the ImageSetConfiguration
We will create a dedicated directory under `/mnt/low-side-data` and write our configuration file there.

```bash
cd /mnt/low-side-data/
mkdir gitops-operator
cd /gitops-operator
vim imageset-config.yaml
```

Paste the following content into `imageset-config.yaml`:
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

