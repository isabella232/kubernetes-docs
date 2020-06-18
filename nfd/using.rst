Using Node Feature Discovery in a Kubernetes Cluster
#####################################################

Before using Kubernetes Node labels that published by NFD, NFD itself should be deployed through standard Kubernetes resource DaemonSet or Job. Here, we recommend deploying NFD as a Kubernetes DaemonSet. this ensures that all nodes of the Kubernetes cluster run NFD, and, new nodes get automatically labeled as soon as they become schedulable in the cluster. Meanwhile, as a DaemonSet, NFD runs in the background, re-labeling nodes every 60 seconds (by default) so that any changes in the node capabilities are detected.
For quick start, you can use the provided template specs to deploy the NFD release image with the default configuration in the default namespace.

Deployment script
====================

Start with an already initialized Kubernetes cluster, if the Kubernetest cluster is not ready, please follow up the `Setup Instructions <https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/>`__

Step 1 - Download and check the content of NFD yaml file
--------------------------------------------------------

The `Node feature discovery project <https://github.com/kubernetes-sigs/node-feature-discovery>`__ provides sample template for quick test.

.. code-block:: bash
   
   curl https://raw.githubusercontent.com/kubernetes-sigs/node-feature-discovery/master/nfd-daemonset-combined.yaml.template

Step 2 - Apply Daemonset resource
---------------------------------

.. code-block:: bash

   kubectl create -f https://raw.githubusercontent.com/kubernetes-sigs/node-feature-discovery/master/nfd-daemonset-combined.yaml.template


.. note:: 

   Once applied above DaemonSet resource file to a real Kubernetes cluster, it will create a service account, clusterrole, and clusterrolebinding.

Step 3 - Show the labels of the node
------------------------------------

.. code-block:: bash

   kubectl get no -o json | jq '.items[].metadata.labels'
   kubectl get no -o json | jq '.items[].metadata.annotations'
   kubectl describe sa/nfd-master clusterrole/nfd-master clusterrolebinding/nfd-master

Here is `simple demo <https://gitlab.devtools.intel.com/ssp-demos/kubernetes/node-feature-discovery-simple-demo#screenshots>`__ . 
 
NFD software itself consists of two components: nfd-master and nfd-worker

-  nfd-master is responsible for labeling Kubernetes node objects
-  nfd-worker is for detecting features and communicates them to nfd-master.
   One instance of nfd-worker is supposed to be run on each node of the cluster

For more detail of deployment, please `see <https://github.com/kubernetes-sigs/node-feature-discovery#usage>`__ or the `live demo <https://asciinema.org/a/247316>`__ .

Usage of the node labels
========================

Now, it’s time to describe using Labels to schedule pods, these labels can be used for placing hard or soft constraints on where specific pods should be run. 

There are two methods: `nodeSelector <https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector>`__ and `nodeAffinity <https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity>`__ .

NodeSelector is a simple and limited mechanism to specify hard requirements on which node a pod should be run. nodeSelector contains a list of key-value pairs presenting required node labels and their values. A node must fulfill each of the requirements, that is, it must have each of the indicated label-value pairs for the pod to be able to be scheduled there.
For example, it is possible to ensure that ML jobs are scheduled on AVX-512 nodes, so the pod yaml specifically requests AVX-512 using the following:

.. code-block:: yaml

    apiVersion: v1 
    kind: Pod 
    metadata:  
        name: node-selector-example 
        spec:  
       nodeSelector: feature.node.kubernetes.io/cpu-cpuid.AVX512BW: 'true'
       containers: ML-demo

NodeAffinity provides a much more expressive way to specify constraints on which nodes a pod should be run. It provides a range of different operators to use for matching label values (not just “equal to”) and allows the specification of both hard and soft requirements (i.e. preferences).
Currently, the `node feature discovery <https://github.com/kubernetes-sigs/node-feature-discovery>`__ provided framwork and reference implement for exposing hardware platform features through note labels. In most cases, per products requirement, user needs to custom-build an NFD version to meet specific demands, this is not difficult

Customization 
=================

Clone Node feature discovery code base for extention and customization, then generate customized docker images, push them into your registry. 

.. code-block:: bash

    git clone https://github.com/kubernetes-sigs/node-feature-discovery
    cd <project-root>
    make
    docker push <IMAGE_TAG>

After that, refer to `nfd-daemonset-combined.yaml.template <https://github.com/kubernetes-sigs/node-feature-discovery/blob/master/nfd-daemonset-combined.yaml.template>`__
to write a new deployment resource file so that your customized NFD docker images be used. 

For instructions on building source code of the `Node feature discovery project <https://github.com/kubernetes-sigs/node-feature-discovery>`__, please follow up: 

`Build source code <https://github.com/kubernetes-sigs/node-feature-discovery#building-from-source>`__

There is also a way to write a device plug-in to expose additional hardware features, like the 3rd acceleration cards. Please refer to the `open source project <https://github.com/intel/intel-device-plugins-for-kubernetes>`__.
