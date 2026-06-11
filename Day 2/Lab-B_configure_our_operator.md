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

<br>

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

<br>

**2.b. CSV (Cluster Service Version)**
Run a similar command to view the ClusterServiceVersion (CSV) objects:
```bash
$ oc get csv -A
```

<br>

**2.c. OperatorGroup**
Run a similar command to see the OperatorGroup objects:
```bash
$ oc get og -A
```

<br>

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


<br><br>

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
