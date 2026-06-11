# 🚀 Day 1: The Foundation - Architecture & Installation

Welcome to Day 1! 🎉 
Today is all about understanding what lives "under the hood" and how a cluster is born. Let's build a strong foundation!

<br>

## 🎯 Today's Goals
*   Understand OpenShift Architecture.
*   Perform an installation in a disconnected environment.

<br><br>
> [!NOTE]
> ## 🖥️ TODAY'S LAB IS LIVE IN THE WORKSHOP UI
> 
> Today, the entire lab environment and all cluster installation exercises are hosted directly within our **Workshop UI**.
> 
> 👉 **Please log in to your provided lab link and follow the guide instructions.**

<br><br><br><br>


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
