Node Feature Discovery (NFD)
##########################################

Kubernetes is becoming more and more popular as the first 
Cloud Native Computing Foundation (CNCF) project, for managing containerized 
workloads and services, and there are lots of productions based on Kubernetes 
in cloud industry today. However, there is no way to identify hardware 
capabilities or configuration Inability for workload to request certain 
hardware feature, especially non-allocable resources, 
like Intel Advanced Vector Extensions (Intel AVX), 
Intel Resource Director Technology (Intel RDT) in Intel platform. 
So, intel and open source community established a project 
named "Node Feature Discovery", the target is to detect resources on each 
node in a Kubernetes cluster and advertises those features so that matching 
workload to platform capabilities. 

.. toctree::
   :maxdepth: 2

   overview
   enabling
   using

Source code of the Node Feature Discovery
=========================================

Node Feature Discovery `Github Repository <https://github.com/kubernetes-sigs/node-feature-discovery>`__
