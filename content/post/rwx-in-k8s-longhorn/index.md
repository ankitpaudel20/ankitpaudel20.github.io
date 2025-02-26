---
title: "Rwx Volume in k8s: Adventure with Longhorn" # Title of the blog post.
date: 2025-02-06T00:44:36+05:45
description: "Lessons learnt with using Lonhorn for setting up RWX volumes in k8s cluster." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: true # Controls if a table of contents should be generated for first-level links automatically.
summary: "Intro to solutions for RWX volume in k8s cluster. The experience with using longhorn for the job."
usePageBundles: true # Set to true to group assets like images in the same folder as this post.
featureImage: "longhorn-k8s-main-image.png" # Sets featured image on blog post.
thumbnail: "thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - kubernetes
tags:
  - kubernetes
  - storage
---

# Preface:
- This Blog talks about my experience with setting up and using Longhorn for our RWX needs.
- We decided to go against longhorn due to it being too unstable in rapidly changing cluster
- The final choice we landed on was using a VM to host nfs off cluster with proper hardning.

# Problem:

- Our setup required sharing large data between kubernetes pods.
    - Much like map reduce, lot of tasks would create a large number of files, save it to nfs, and one last task that collated all those files, ran some logic to make sense out of it and return.
    - We used Celery’s canvas for this.
- This required us to setup a simple nfs server (How to for this is available all over the internet)
    - This involved, a deployment with single pod running nfs-server, a pv using nfs csi native with k8s and finally a rwx pvc for a pod to use.
- This made nfs our single point of failure, random nfs-restarts would cause badly configured application writing to it, error out with `PermissionError` .
    - This happened in our python application.
- This issue would only arise when the existing pod of nfs-server switches nodes.
    - This can happen in any scenario, during cluster scaling events or during node upgrade period.

# The Solution:

- Make highly available RWX Storage that can survive node unavailability.
- Searched a bit around internet, they were suggesting us to use managed nfs solution provided by cloud provider.
    - GCP’s offering `filestore` requires minimum size of the storage be 1TB which was a lot. We needed around 20GB of storage at max
    - AWS has its offering with EFS but we did not have luxury of using it.
- Stumbled upon a few implementations:
    - OpenEBS:
        - Good, but its rwx offering was still not GA.
    - Rook:
        - Best, but was too complex for our usecase. Ceph itself is too complex thus decided to not include its complexity for a small team like us. Suspected it as a future tech debt, requiring ceph and rook specific engineer to maintain it in long run.
    - Longhorn:
        - Suited our case. RWX offering at GA. Not too complex architecture.
- Hence we chose Longhorn.

# Longhorn

- Very easy to setup, finicky sometimes.
- Major issue, clusters with spot instances are on high chances of faulting.
- Used following helm-chart values for fast fail-over and good resiliency:
    
    ```yaml
    defaultSettings:
      createDefaultDiskLabeledNodes: false
      persistence.defaultClass: false # set this to false in prod
      guaranteedInstanceManagerCPU: 5
      kubernetesClusterAutoscalerEnabled: true
      autoSalvage: true # tries to recover from multiple node failures
      nodeDownPodDeletionPolicy: delete-both-statefulset-and-deployment-pod
      nodeDrainPolicy: block-for-eviction-if-contains-last-replica
      replicaAutoBalance: best-effort
      replicaReplenishmentWaitInterval: 30
      rwxVolumeFastFailover: true
      fastReplicaRebuildEnabled: true
      snapshotDataIntegrity: "fast-check" # this is required for fast replica rebuild.
    persistence:
      defaultClassReplicaCount: 3
      defaultDataLocality: disabled
    longhornManager:
      log:
        format: json
    longhornDriver:
      log:
        format: json
    ```
    
- Uses a open source project called NFS-Ganesha for its rwx volume.
    - This allows nfs to run in userspace without nfs’s kernel module.

## **Creating a new RWX PV ( Its not provided by default during installation)**

- First create a volume from LonghornUI or from longhorn cli application.
- Create a storage class that supports RWX and uses longhorn csi driver:
    
    ```yaml
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: longhorn-nfs
    provisioner: driver.longhorn.io
    allowVolumeExpansion: true
    reclaimPolicy: Delete
    volumeBindingMode: Immediate
    parameters:
      numberOfReplicas: "3"
      staleReplicaTimeout: "2880"
      fromBackup: ""
      fsType: "ext4"
      nfsOptions: "vers=4.1,noresvport,softerr,timeo=600,retrans=5"
    ```
    
- Create a `Longhorn Volume` from Longhorn UI or from Longhorn CLI, set the replicas to at least 3, Set `Access Mode` to `ReadWriteMany` ,Set `Data Locality`  to `best-effort` .
- Go to the Volume Created in the UI, there should be a dropdown besides it which has option saying something like `Create PV/PVC` , click that.
- Set `Storage Class Name` to `longhorn-nfs` , `PVC Name` , `Namespace` to namespace you want to bind the volume to .

## Solving Basic Issues:

- Once a volume faults, its going to be hard to recover it
    - Manual salvage. [How to](https://docs.uipath.com/automation-suite/automation-suite/2022.4/installation-guide/all-longhorn-replicas-are-faulted)
- Okay when failure rate is not that high, when used with spot instances, I experienced random volume being faulted overnight.

## **How to recover from faulted volume:**

- First check what is the actual issue.
- Case 1: When there are a few replicas online but replication sequence is not starting.
    - Navigate to the replica that is faulted and is having hard time to get out of the quorum, set the `faultedAt` part to empty string, apply it and let the controller handle the retry.
- Case 2: It says instance manager not running in the node running a pod that is trying to attach the pvc. Longhorn ui shows it being attached to some pod that does not exist anymore.
    - Try to evict the pod from that node. Just deleting the pod might not work, you can use one a kubectl extension available in krew named `evict-pod` use it. If the pod is controlled by some other resource, you can try to scale down it to 0 and then to rescale back.
    - If that does not work, you might need to force detach the volume. It might not be possible from Longhorn UI, but you can edit the yaml of longhorn volume, set the nodeID or some other label that indicates the volume attachment to node to an empty string.
- Case 3: The volume itself is faulted, No healthy replica remains.
    - Check if the volume still says attached to non existing node, if that is the case, remove that node from the volume by editing the yaml of the faulted resource.
    - If the volume says its detached and Faulted, you can just reduce the replicas by editing yaml of the volume, make it 1, edit the remaining replica,  set the field containing attached node information and `failedAt` both to an empty string.

# The Conclusion

- Longhorn is not stable enough or is difficult to stabilize it in spot instance cluster.
- If it does not work efficiently in a spot instance cluster, its not reliable enough to ship to production.