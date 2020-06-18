Node Feature Discovery Overview
###############################

Node Feature Discovery (NFD) is a Kubernetes\* add-on that detects and advertises
hardware and software capabilities of a platform that can, in turn, be used to 
facilitate intelligent scheduling of a workload. which can be used to facilitate
improved workload placement based on platform capabilities, NFD is an open source
Kubernetes community project.

The software is available in our `GitHub repository <https://github.com/kubernetes-sigs/node-feature-discovery>`__

In a standard workload deployment, Kubernetes reveals very few details about the
underlying platform to the developer or end-user. This may be a good way for 
general data center using, but in many cases a workload behavior or its performance
may improve by leveraging the platform (hardware and/or software) features.
NFD is defined only to handle non-allocable features, that is, unlimited capabilities 
that do not require any accounting and are available to all workloads. As we known, 
allocable resources that require accounting, initialization and other special handling,
which are presented as Kubernetes Extended Resources and handled by device plugins,
so allocable resources, like GPU, it is out of the scope of NFD.

Detected features
=====================================

CPUID
   Intel processors have a special CPUID instruction for determining the CPU features,including the model and support for instruction set extensions, such as Intel Advanced Vector Extensions (Intel AVX). Certain workloads, such as machine learning, may gain a significant performance improvement from these extensions (e.g. AVX-512). NFD advertises all CPU features obtained from the CPUID information. 

SR-IOV networking
   Single Root I/O Virtualization (SR-IOV) is a technology for isolating PCI Express resources. It allows multiple virtual environments to share a single PCI Express hardware device (physical function, PF) by offering multiple virtual functions (VF) that appear as separate PCI Express interfaces. In the case of network interface cards (NICs), SR-IOV VFs allow direct hardware access from multiple Kubernetes pods, increasing network I/O performance, and making it possible to run fast user-space packet processing workloads (for example, based in Data Plane Development Kit). NFD detects the presence of SR-IOV-enabled NICs, allowing optimized scheduling of network-intensive workloads. 

Intel RDT
   Intel Resource Director Technology (Intel RDT) allows visibility and control over the usage of last-level cache (LLC) and memory bandwidth between co-located workloads. By allowing allocation and isolation of these shared resources, and thus reducing contention, RDT helps in mitigating the effects of noisy neighbors. This provides more consistent and predictable performance which may be essential in meeting Service Level Agreements (SLA), for example. NFD detects the different RDT technologies supported by the underlying hardware platform. 

Intel Turbo Boost Technology
   Intel Turbo Boost Technology accelerates processor performance for peak loads, dynamically overclocking processor cores if they are operating within the power, current, and temperature limits of the processor. This can provide significant performance benefits for CPU-bound workloads. On the other hand, some workloads behave better when this technology has been disabled. NFD detects the state of Intel Turbo Boost Technology, allowing optimal scheduling of workloads that have a well-understood dependency on this technology. 

IOMMU
   An input/output memory management unit (IOMMU), such as Intel Virtualization Technology (Intel VT) for Directed I/O (Intel VT-d) technology, allows isolation and restriction of device accesses. This enables direct hardware access in virtualized environments, highly accelerating I/O performance by removing the need for device emulation and bounce buffers. This can be crucial for I/O heavy workloads in Kubernetes deployments using hypervisor-based container runtimes, such as Kata Containers. NFD detects if an IOMMU is supported by the host hardware platform and enabled in the kernel of the host operating system. 

SSD storage
   Solid state drives (SSD) have a huge performance advantage over traditional rotational hard disks. This may be important for disk I/O intensive workloads. NFD detects the presence of non-rotational block storage on the node, making it possible to accelerate workloads requiring fast local disk access. 
 
NUMA topology
   Non-uniform memory access (NUMA) is a memory architecture where CPU’s memory access times are dependent on the memory location. Access to CPU’s local memory is faster than to non-local memory (local memory of another CPU) which can cause workloads to perform poorly if not properly designed for NUMA systems. On the other hand, some highly NUMA-aware applications may experience negligible performance penalties. NFD detects the presence of NUMA topology, making it possible to optimize scheduling of applications based on their NUMA-awareness. 

Linux\* kernel
  Some specific workloads may be highly dependent on the kernel version of the underlying host operating system. For example, some kernel features may be required to be able to run an application, or, they provide measurable performance benefits. NFD detects the kernel version and advertises it through multiple labels, allowing the deployment of workloads with different granularity of kernel version dependency. 

PCI
   Detecting the presence of compatible PCI hardware devices is beneficial for some workloads. For example, Kubernetes device plugins need to be deployed only on nodes that have hardware that the device plugin manages. NFD detects PCI devices, allowing optimized scheduling of workloads dependent on certain PCI devices.

Supported features
==================

.. figure:: /_images/image001.png

   Feature list
