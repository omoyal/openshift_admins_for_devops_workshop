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
ssh highside

oc login https://api.disco.lab:6443 --username kubeadmin
(# The kubeadmin password in: ' /mnt/high-side-data/auth/kubeadmin-password ')
oc new-project dev-app-project
oc new-project prod-app-project
```

---
<br><br>


## Part 2: Creating the Local Identity Provider (IdP)

We will deploy our user database directly into OpenShift using a Kubernetes Secret. Both users (`disco-admin` and `disco-dev`) are configured with the same password: **`Disco123!`**

### 1. Create the HTPasswd File
Run the following commands to write the pre-generated secure password hashes into a local temporary file:

```bash
echo "disco-admin:{SHA}Uo3KBq9vhu4j2V0klNxVs38fAR4=" > /tmp/htpasswd
echo "disco-dev:{SHA}Uo3KBq9vhu4j2V0klNxVs38fAR4=" >> /tmp/htpasswd
```

### 2. Inject the File as an OpenShift Secret
Now, load the file into OpenShift's core configuration namespace:

```bash
oc create secret generic htpasswd-secret --from-file=htpasswd=/tmp/htpasswd -n openshift-config
```

### 3. Update the Cluster OAuth Configuration
Apply the following configuration to patch the global OAuth resource and register our new provider:

```bash
cat << 'EOF' | oc apply -f -
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

### 4. Force Authentication Pods Refresh (CRITICAL)
Whenever a secret's content is modified, we must force the authentication deployment to restart so it reads the new user database:

```bash
# Trigger a rolling restart for the OAuth pods
oc rollout restart deployment/oauth-openshift -n openshift-authentication

# Monitor the rollout status until it completes successfully
oc rollout status deployment/oauth-openshift -n openshift-authentication
```
*(Wait until you see: `deployment "oauth-openshift" successfully rolled out`)*


---

### Now Check the new users  
Check the Disco-admin:
```bash
# Kubeadmin logout
oc logout

# Login with Disco-admin
oc login https://api.disco.lab:6443 -u disco-admin
< Password is: Disco123! >

# verify
oc whoami

# lets check if you have any permissions in cluster
oc get pods
oc get nodes
```


Check the Disco-dev:
```bash
# Disco-admin logout
oc logout

# Login with Disco-dev
oc login https://api.disco.lab:6443 -u disco-dev
< Password is: Disco123! >

# verify
oc whoami

# lets check if you have any permissions in cluster
oc get pods
oc get nodes
```

---








<br><br><br>
## Part 3: Enforcing RBAC (Permissions)

Now that our identity provider is live, let's configure cluster and namespace-level permissions for our new users.

<br>

### 5. Assign Cluster Admin Rights
Give `disco-admin` full administrative access to the entire cluster:

```bash
oc adm policy add-cluster-role-to-user cluster-admin disco-admin
(# explain: 'add-cluter-role-to-user'Automaticly create a ClusterRoleBinding from user to ClusterRole: 'Cluster-admin')
```

<br>

### 6. Assign Admin-Namespace-Level Developer Rights
Give `disco-dev` developer permissions **only** inside the development project:

```bash
oc adm policy add-role-to-user admin disco-dev -n dev-app-project
```

---


<br><br>


## Part 4: Verification Challenge 🧪

Let's verify that our authentication and authorization boundaries work perfectly.

### 1. Test the Developer Login
Log in as the developer and attempt to interact with both projects:

```bash
# Log in
oc login https://api.disco.lab:6443 -u disco-dev
# < Password: Disco123! >

# Test Dev project (Should succeed)
oc get pods -n dev-app-project

# Test Prod project (Should fail with Forbidden / Access Denied)
oc get pods -n prod-app-project
```

<br>

### 2. Test the Admin Login
Switch back to the administrator account to confirm cluster-wide access:

```bash
oc login https://api.disco.lab:6443 -u disco-admin
# < Password: Disco123! >

# Check Dev ns
oc get pods -n dev-app-project

```

**Congratulations! Your first security milestone is complete! 🎉**
