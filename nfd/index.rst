Node Feature Discovery (NFD)
##########################################

Kubernetes\* is becoming more and more popular as the first
Cloud Native Computing Foundation (CNCF) project, for managing containerized
workloads and services, and there are already lots of products based on
Kubernetes in the industry today. However, there is no way to identify
hardware capabilities or configurations. There is also no way for a workload
to request a specific hardware feature, especially non-allocable resources,
like Intel® Advanced Vector Extensions (Intel® AVX) or
Intel Resource Director Technology (Intel® RDT) on Intel platforms.
So, Intel® and the open source community established a project
named "Node Feature Discovery", the goal is to detect resources on each
node in a Kubernetes cluster and advertise those features to match the
workload to platform capabilities.

.. toctree::
   :maxdepth: 2

   overview
   enabling
   using

Source code of the Node Feature Discovery
=========================================

Node Feature Discovery `Github Repository <https://github.com/kubernetes-sigs/node-feature-discovery>`__
