# Day 3 Lab 1 - Securing the Gates (OAuth & RBAC)

Welcome to Day 3!  
In this first exercise, we will take on the role of an SRE hardening our cluster.  
We will connect OpenShift to a local Identity Provider using an `htpasswd` configuration, create enterprise users, and set up strict access control rules (RBAC).

> [!IMPORTANT]
> **Our Mission:**
> Create a dual-projecrs (NameSpaces) setup (`dev` and `prod`), onboard an Admin and a Developer, and ensure each has exactly the permissions they need—no more, no less.


> [!IMPORTANT]
> Just like the previous labs, we will walk through this process together, step-by-step. 
> Take your time to understand each step!

**Good luck!** 🚀


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




## Part 1: Setting Up the Environments
Let's start by creating our isolated Projects (NameSpaces).  
VERIFY you are on **Highside Server** (🟠 Orange prompt) !

```bash
oc login https://api.disco.lab:6443 --username kubeadmin
(# The kubeadmin password in: ' /mnt/high-side-data/auth/kubeadmin-password ')
oc new-project dev-app-project
oc new-project prod-app-project
```

---
<br><br>
## Part 2: Creating the Local Identity Provider (IdP)

We will deploy our user database directly into OpenShift using a Kubernetes Secret. Both users (`disco-admin` and `disco-dev`) are pre-configured with the same password: **`Disco123!`**

### 1. Create the HTPasswd Secret
Run the following command to create the secret configuration file directly inside the cluster:

```bash
cat <<EOF | oc apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: htpasswd-secret
  namespace: openshift-config
type: Opaque
stringData:
  htpasswd: |
    disco-admin:{SHA}W6ph5Z1Cg8YJPhceX/+bS7I6Eic=
    disco-dev:{SHA}W6ph5Z1Cg8YJPhceX/+bS7I6Eic=
EOF
```

### 2. Update the Cluster OAuth Configuration
Now, let's patch the global OAuth object to tell OpenShift to use this secret as our corporate login provider:

```bash
cat <<EOF | oc apply -f -
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: local-htpasswd-idp
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpasswd-secret
EOF
```

> [!TIP]
> Wait a minute or two for the Authentication Cluster Operator (`oc get co authentication`) to refresh and load the new provider.
> 






---

## Part 3: Enforcing RBAC (Permissions)
Now that our users can log in, let's make sure they can only touch what they are authorized to.

### 4. Assign Cluster Admin Rights
We want `disco-admin` to have full control over the entire cluster:

```bash
oc adm policy add-cluster-role-to-user cluster-admin disco-admin
```

### 5. Assign Namespace-Level Developer Rights
We want `disco-dev` to be able to work and deploy applications **only** inside the development project:

```bash
oc adm policy add-role-to-user edit disco-dev -n dev-app-project
```

---

## Part 4: Verification Challenge 🧪

To verify your work, open a new terminal tab or log out of your current session, and try the following:

1. Log in as the developer: `oc login -u disco-dev`
2. Try to view pods in the dev project: `oc get pods -n dev-app-project` *(Should succeed)*
3. Try to view pods or create resources in the prod project: `oc get pods -n prod-app-project` *(Should fail with an Access Denied / Forbidden error!)*

**Congratulations! Your first security milestone is complete! 🎉**
