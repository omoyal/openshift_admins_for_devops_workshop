# Lab 5: Advanced Observability and Proactive SRE 

This lab focuses on transforming raw cluster metrics into actionable insights and proactive SRE operations.  
You will configure the OpenShift Monitoring stack to enable **User Workload Monitoring (UWM)**, allowing developers to observe their applications natively.  
You will then simulate infrastructure pressure to validate your monitoring pipelines and practice data-driven troubleshooting from the developer's perspective.


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


---

## Part 1: Self-Guided Tour of Cluster Dashboards

As infrastructure admins, your starting point during an anomaly is the Observe Tab.

1. Log in to the OpenShift Web Console and switch to the **Administrator** perspective.
2. Navigate to the monitoring menu on the left: **Observe** -> **Dashboards**.
3. **Use Your Head:** Play with the Dashboard selection dropdown at the top of the screen. Find the specific dashboard for **Compute Resources / Cluster**.
4. **Food for Thought:** What is the current Memory utilization percentage for your entire cluster?

---

## Part 2: Enabling User Workload Monitoring (UWM)

By default, OpenShift only monitors core system components. To allow developers to see metrics for their applications, we must enable the dedicated component within the Cluster Monitoring Operator's central configuration.

1. You need to create or edit a configuration file (ConfigMap) named `cluster-monitoring-config` within the `openshift-monitoring` Namespace.
2. Here is the file template. **Fill in the missing line** (instead of the `_______`) to enable the feature (use an appropriate boolean value):

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |
    enableUserWorkload: _______

```

3. Apply the file to the cluster:
```bash
oc apply -f cluster-monitoring-cm.yaml

```


4. **Verification:** Run the following command and watch for the new pods being created. Why does the Operator create an entirely new Namespace named `openshift-user-workload-monitoring`?
```bash
oc get pods -n openshift-user-workload-monitoring -w

```


> 💡 **Admin Tip:** It may take up to 2 minutes for the new User Workload Prometheus pods to transition to the `Running` and `Ready` state.



---

## Part 3: Creating a Developer Project and Stressing Resources (Air-Gapped)

Now we will impersonate a developer, create a new project, and deploy an application that forces the node to allocate high resources (generating a metric Spike). Since we are working in an air-gapped environment, we will use the CLI image already available within the cluster's internal Registry.

1. Create a new Namespace for the application:
```bash
oc new-project day5-apps

```


2. Apply the following Deployment. It runs an infinite loop of mathematical calculations (`md5sum /dev/zero`) that will immediately max out a full CPU core:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: infra-stresser
  namespace: day5-apps
spec:
  replicas: 1
  selector:
    matchLabels:
      app: stresser
  template:
    metadata:
      labels:
        app: stresser
    spec:
      containers:
      - name: cpu-burner
        image: image-registry.openshift-image-registry.svc:5000/openshift/cli:latest
        command: ["/bin/sh", "-c", "md5sum /dev/zero"]
        resources:
          limits:
            cpu: "1"
            memory: "200Mi"
          requests:
            cpu: "500m"
            memory: "100Mi"

```

3. Apply the deployment:
```bash
oc apply -f stress-app.yaml

```



---

## Part 4: Reviewing Results from the Developer Perspective (Developer View)

Let's see how the feature we enabled in Part 2 helps developers troubleshoot performance issues on their own, without opening support tickets for the infra team.

1. Return to the OpenShift Web Console and switch the perspective in the top-left corner from **Administrator** to **Developer**.
2. Ensure that the project you created is selected in the Project dropdown: `day5-apps`.
3. Navigate to: **Observe** -> **Metrics**.
4. **Critical Thinking Task:** Switch to the **Dashboard** tab (within the developer's Observe window) and select the **Pod** dashboard.
5. Look at the **CPU Usage** graph:
* Do you see the Spike in the graph?
* To what exact value (in cores or millicores) did the pod reach?
* Why did it stop exactly at that value and not continue to rise? (Hint: Look at the `limits` defined in the YAML file).



```

```
