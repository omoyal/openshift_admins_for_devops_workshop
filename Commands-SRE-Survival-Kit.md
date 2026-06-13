# 🛠️ OpenShift SRE & Admin Survival Kit

This cheat-sheet compiles the most essential and frequently used commands throughout this workshop. Use it to audit cluster health, manage your air-gapped registry, explore resources, and debug issues.

---

## 🗂️ 1. Discovery & Learning the API
Before running a command, use these to discover what resources exist in the cluster and how to configure their YAML structure.

```bash
# Get the global help menu or help for a specific subcommand
oc help
oc help get

# List all available resource types (CRDs) supported by the cluster API
oc api-resources

# Inspect the documentation and fields of a specific resource (Acts as an offline man-page)
oc explain pod
oc explain networkpolicy.spec
```

---

## 🛡️ 2. Core Cluster Administration (Health & Status)
The "Holy Trinity" of cluster auditing for an OpenShift Administrator. Run these first when checking if the infrastructure is stable.

```bash
# Check the status, roles, and Kubernetes version of all cluster nodes
oc get nodes

# View the health status of all OpenShift core operators (Ensure all are AVAILABLE=True)
oc get co

# Audit Machine Config Pools to see if nodes are updating or degraded after a change
oc get mcp
```

---

## 📦 3. Air-Gapped Operations (`oc-mirror`)
Essential commands used during Day 1 to mirror images, operators, and platform releases to your disconnected registry.

```bash
# Dry-run or execute image mirroring based on your Imageset configuration
oc mirror --config=imageset-config.yaml file://mirror-v2

# Push mirrored images from a local folder to your target enterprise registry
oc mirror --from=file://mirror-v2 docker://[registry.example.com/ocp4](https://registry.example.com/ocp4)

# List available operator channels and catalogs to include in your config
oc mirror list operators
oc mirror list operators --catalog=registry.redhat.io/redhat/redhat-operator-index:v4.17
```

---

## 🎯 4. Project & Context Management
Commands to navigate between different namespaces and verify project states.

```bash
# View details of a specific project
oc get project <project-name>

# Switch your active terminal context to a specific project (No need to pass -n every time)
oc project <project-name>
```

---

## 🔍 5. Resource Inspection & Troubleshooting
When an application fails, use these commands to locate the faulty resource and dig into its lifecycle events.

```bash
# List all standard resources (Pods, Services, Deployments, etc.) across ALL namespaces
oc get all -A

# List all standard resources ONLY in your currently active namespace
oc get all

# Deep-dive into a specific Pod's events, status, lifecycle, and error messages
oc describe pod <pod-name>

# View real-time logs of a container to debug application-level failures
oc logs <pod-name>
oc logs <pod-name> -c <container-name> --tail=100 -f
```

---
*💡 **Tip:** Keep this page open in a side-tab throughout the practical labs!*

