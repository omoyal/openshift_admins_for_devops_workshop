# Day 2 Lab 1 - Our First Operator

Hey,
In this lab, we will simulate a very common real-world scenario: bringing an external operator into our environment and installing it inside the cluster. 
We will do this together, step-by-step.

Please make sure to read and understand the commands before running them, so you know exactly what they do...

Of-cource, If you have any questions, don't hesitate to ask the instructor.

Best of luck! 💪


<br> <br> <br>

1. go to the Jumo server (blue) and lets download the Openshift-gitops-operator in latest version from the internet via oc-mirror


1.a.
  create imagesetconfiuration in dedicated dir under /mnt/low-side-data: 
  (Verify you are in Jump Server)
  $ cd /mnt/low-side-data/
  $ mkdir gitops-operator
  $ cd /gitops-operator
  $ vim imageset-config.yaml :
 
Yaml:
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


1.b.
 Downloading the operator via oc-mirror:
 
 this is the command:
 $ oc mirror -c imageset-config.yaml file:///mnt/low-side-data/gitops-operator --v2


1.c.
wait the oc-mirror say: " 👋 Goodbye, thank you for using oc-mirror "




2. after the operator downloded we need the move the tar output to our Air-gap env:
  this is the command:
  rsync -avP /mnt/low-side-data/gitops-operator highside:/mnt/high-side-data/

  wait the copy done



  -------------------------------
  -- now we go to Air-gap env
  -------------------------------
 

  3. connect to Highside server (in AirGap env), investing the our new tar
     $ ssh highside
     $ cd /mnt/high-side-data/gitops-operator

Verify again you are on Highside server (Orange)


  4. use oc-mirror to push new operator to our Registry (quay.io) [maybe you need to login]
     $ oc mirror -c <image_set_configuration> --from file:///mnt/high-side-data/gitops-operator docker://$(hostname):8443 --v2

     wait the oc-mirror say .....

5. lets invesigate the output from this action
   $ ls -lah
   $ ls -lah working-dir/cluster-resources/
   read the files...


6. before yo apply our new file on Openshift we need to Disabeld the Current catalogSources in the our cluster
   (The logic is: since OpenShift doesn't know whether it has internet access or not, it will constantly throw errors trying to pull the available indexes from the internet.             Therefore, we disable the default ones.)
   
   
   6.a login to the cluster from the cli:
   $ oc login -u kubeadmin https://api.disco.lab:6443
   (you can find your kubeadmin paswword in highside-server: ' /mnt/high-side-data/auth/kubeadmin-password ')

   6.b Disabled the default catalogSources with this command:
   $ oc patch OperatorHub cluster --type merge -p '{"spec": {"disableAllDefaultSources": true}}'



8. lets deploy the k8s-object-mirroring to our cluster 
   $ oc apply -f /mnt/high-side-data/gitops/working-dir/cluster-resources/


9. go to the terminal and prompt:
  $ oc get mcp
  $ oc get pods -n openshift-marketplace

# because the IDMS need to update each node in cluster the MCP (MachineConfigPool) is updated
# you can see in the ns openshift-marketplace have a new pod - this pod is a our new catalogSource index
# go to the Web-openshift and under Operators -> operatorHub -> you can see the our new Operator

Congradulations! 

ALL Deserved!!!!!

  
