Intel® Device Plugins for Kubernetes\*
##########################################

With deep learning and artificial intelligence being used in many kinds of
areas, heterogeneous computing has become quite hot, so various hardware
providers in the industry have developed heterogeneous computing cards or
hardware accelerators to support those application scenarios. For instance,
Intel® developed several hardware devices to meet those requirements: Intel®
QuickAssist Technology (Intel® QAT), Intel® FPGA, Intel® Artificial
Intelligence and Intel® Server GPU.
The intent of this documentation is to help developers/users understand how to
enable and use those Intel® hardware devices in Kubernetes cluster.

.. toctree::
   :maxdepth: 2

   gpu
   fpga
   qat
   vpu

Device Management in Kubernetes
===========================================

Usually once a device, device driver, and related develop kits are installed in system, it is simple to use those heterogeneous cards or devices in host system and traditional docker container environment with *--device* and  *--volume* parameter running, in docker environment, this is docker container mapping mechanism to use host system resource directly in containers through *--device*. but in a kubernetes cluster environment, resources of these heterogeneous computing card/device to be bond with container needs to be managed and allocated by kubernetes, that means after installing those heterogeneous computing cards in each computing node, We still need to have a mechanism to tell the cluster that a certain kinds of heterogeneous card and the number of cards have been installed on a computing node, and provides methods so that the cluster can manage and allocate this heterogeneous computing card resource when application needs placing. in order to have device resource manage and allocate capability in kubernetes and adoption for devices venders, the Kubernetes is designed to have 2 built-in modules, **Extended Resource** and **Device Plugin**, the specific device plugins developed by device vendors will implement scheduling from a device cluster to working nodes, and then bind devices with containers.

Extended Resource
    This is a custom resource expansion method, it belongs to node level API.  It can be used independently. once installed the device and device drivers in node. the developer needs to report the names and the total number of the device resources to the kubernetes API server, the scheduler will increases or decreases the number of available resources based on the creation or deletion in the resource pool and determines nodes that satisfy resource requirements during scheduling. the increment and decrement of Extended Resource must be integers. for example, you can allocate 1 GPU but cannot allocate 0.5 GPUs.  it is simple to report extended resources and update status through a PACTH API, and the PACTH API operation can be completed through a simple curl command as well.

Device Plugin
    It provides a general device plugin mechanism, called framework and standard device API interface. device vendors can expand devices such as the QAT, FPGA, GPU, and VPU by implementing APIs, without modifying the Kubelet main code. the device plugin mechanism is not complex, it's work mechanism mainly includes 2 parts, One is for device resource reporting when start-up, another is for scheduling and allocating when application needs to access.


Device Plugin Working Mechanism
=========================================
Today, most equipment vendors are using Device Plugin mechanism to implement their specific device plugin to be integrated into kubernetes cluster smoothly without modifying kubernetes code, with this way to achieve the equipment scheduling and life cycle management, before knowing how to use intel device plugins in kubernetes, let's understand device plugin internal working mechanism together firstly.
Device Plugin is a simple gRPC server that implements the methods ``ListAndWatch`` and ``Allocate``, then listens the Unix sockets under ``/var/lib/kubelet/device-plugins/``, such as ``/var/lib/kubelet/device-plugins/intel-qat.sock``.

.. code-block:: python

    service DevicePlugin {
        // returns a stream of []Device
        rpc ListAndWatch(Empty) returns (stream ListAndWatchResponse) {}
        rpc Allocate(AllocateRequest) returns (AllocateResponse) {}
    }

ListAndWatch()
    It reports device resources and provides health checking of device. when the device is unhealthy it will be reported to kubelet and the device plugin manager in Kubelet  will remove it's device ID from the schedulable list.
Allocate()
    It is called by device plugin when deploying a specific container application. the core parameter is the device ID, then the returned parameter is the corresponding device path, driver directory, environment variables which  are required when launching the container application, let's see the detail of the process below:


Resource Reporting and Monitoring
--------------------------------------
For each hardware device in kubernetes cluster, there must be a corresponding device plugin to manage. and these device plugins connect the device plugin manager in kubelet through gRPC as the client, the device plugin reports device name and it's UNIX socket API version to kubelet.

.. figure:: /_images/device-plugin-1.png

generally speaking, the whole process is divided into four steps. the first three steps are running on the nodes. the fourth step is the interaction between kubelet and API server.  see below:

The first step is to register device plugin. The kubelet exports a ``Registration`` gRPC service:

.. code-block::python

    service Registration {
        rpc Register(RegisterRequest) returns (Empty) {}
    }

A device plugin can register itself with the kubelet through this gRPC service. Kubernetes needs to know which device plugin to interact with. This is because there may be multiple devices on a node. Device plugin is required to report three things to kubelet as a client. who am I, where am i and the interaction protocol.  see below registration info

- The  name of its Unix socket.
- The  Device Plugin API version against which it was built.
- The Resource Name to be advertised. Here Resource Name needs to follow the `extended resource naming scheme <https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#extended-resources>`__ as vendor-domain/resource type.


The second step is to start a service as a gRPC server. then this service as this server is ready to be accessed by kubelet, the listening address and protocol API version have been exposed to kubelet in the first step

the step 3 is that kubelet to establish a long connection for listandwatch of device plugin to discover the device ID and the health status of the device. when device plugin detects that a device is unhealthy, it will report to kubelet to be aware.

In the last of 4 steps, the device plugin sends the kubelet the list of devices it manages, and the kubelet is then in charge of advertising those resources to the API server as part of the kubelet node status update.

Note:  kubelet only reports the number of specific device  to  API server. the device plugin manager of kubelet itself will save the device ID list of the device and use it for specific device allocation. for the kubernetes scheduler, which does not know the device ID list of the device. It only knows the number of device

Pod Scheduling and Device Resource Allocation
----------------------------------------------

When a container application wants to use a device, it only needs to declare the device resource and the corresponding quantity in the *limits* section of yaml file in pod's resource (for example intel.com/gpu:

.. figure:: /_images/device-plugin-2.png

then Kubernetes scheduler will find the right node  for placing , after that, scheduler will reduce 1 from the saved number of available device and complete the binding pod with node.
After binding successfully, the kubelet running on corresponding node will be responsible for launching the container application. and finds the requested device resource by the pod's container, it will call its internal device plugin manager to get an available device ID from it's device ID list and send this allocated device ID as the parameter to local device plugin service,
After receiving the allocated device ID, device plugin will access  device driver for getting the device Path, driver directory and environment variables of this device , then return them to kubelet as the allocate response.
Once the device path and driver directory information be returned to kubelet, kubelet will perform the operation of assigning device to the container runtime as container launching parameters, after complete device mapping and volume mounting when start the container application. you can check if the device is available to be accessed in container, finally, the device resource allocation for container is end.
above are device plugin API design, working mechanism, and  device plugin lifecycle management

Handling Kubelet Restarts
-----------------------------------------
A device plugin is expected to detect kubelet restarts and re-register itself with the new kubelet instance. In the current implementation, a new kubelet instance deletes all the existing Unix sockets under ``/var/lib/kubelet/device-plugins`` when it starts. A device plugin can monitor the deletion of its Unix socket and re-register itself upon such an event.

Device Plugin Deployment
=========================================
Typically, You can deploy a device plugin as a DaemonSet, or manually.

The canonical directory ``/var/lib/kubelet/device-plugins`` requires privileged access, so a device plugin must run in a privileged security context. If you're deploying a device plugin as a DaemonSet, ``/var/lib/kubelet/device-plugins`` must be mounted as a `Volume <https://kubernetes.io/docs/concepts/storage/volumes/>`__ in the plugin's `PodSpec <https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#podspec-v1-core>`__.

If you choose the DaemonSet approach you can rely on Kubernetes to place the device plugin's Pod onto Nodes, to restart the daemon Pod after failure, and to help automate upgrades.

For example to deploy the DaemonSet of QAT device plugin:

after complete QAT card,QAT driver installation in node and enabling it, to deploying the QAT device plugin involves first the deployment of a `ConfigMap <https://github.com/intel/intel-device-plugins-for-kubernetes/blob/master/deployments/qat_plugin/base/intel-qat-plugin-config.yaml>`__ and the `DaemonSet YAML <https://github.com/intel/intel-device-plugins-for-kubernetes/blob/master/deployments/qat_plugin/base/intel-qat-plugin.yaml>`__

.. code-block:: bash

  $ kubectl create -f deployments/qat_plugin/base/intel-qat-plugin-config.yaml
  $ kubectl create -f deployments/qat_plugin/base/intel-qat-plugin.yaml

This document describes basic knowledge of Kubernetes device plugin design, working mechanism. Please see the rest of the site for details and demos.

Intel Device Plugins for Kubernetes `Github Repository <https://github.com/intel/intel-device-plugins-for-kubernetes>`__
