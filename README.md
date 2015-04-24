#Kubernetes
Kubernetes is an open source system for managing containerized applications
across multiple hosts, providing basic mechanisms for deployment, maintenance,
and scaling of applications.

Kubernetes provides a container management layer over Docker that makes it very
easy to scale a container based application or microservices.

###Pod:

In Kubernetes, rather than individual application containers, pods are the
smallest deployable units that canbe created, scheduled, and managed.
A pod corresponds to a colocated group of applications running with a shared
context.
  
  * serve as units of deployment and horizontal scaling/replication
  * facilitate data sharing and communication among their constituents
  * applications within the pod can see each other's processes
  * applications within the pod have access to the same IP and port space
  * applications within the pod can use SystemV IPC or POSIX message queues
    to communicate
  * applications within the pod share a hostname

When a node dies, the pods scheduled to that node are deleted. Specific pods
are never rescheduled to new nodes; instead, they must be replaced.

###Replication Controller:

A replication controller ensures that a specified number of pod "replicas" are
running at any one time.

*If there are too many, it will kill some.*
*If there are too few, it will start more.*

  * It replaces pods that are deleted or terminated for any reason, such as in
    the case of node failure or disruptive node maintenance, such as a kernel
    upgrade.
  * A replication controller will never terminate on its own, but it isn't
    expected to be as long-lived as services.
  * A replication controller creates new pods from a template, which is
    currently inline in the replicationController object.
  * Subsequent changes to the template or even switching to a new template has
    no direct effect on the pods already created.
  * The population of pods that a replicationController is monitoring is
    defined with a label selector, which creates a loosely coupled relationship
    between the controller and the pods controlled, in contrast to pods, which
    are more tightly coupled.

  * Pods may be removed from a replication controller's target set by changing
    their labels.
####Usage Patterns:
  * Rescheduling
  * Scaling
  * Rolling updates:
    Replication controller is designed to facilitate rolling updates to a
    service by replacing pods one-by-one.
  * Multiple release tracks:
    In addition to running multiple releases of an application while a rolling
    update is in progress, it's common to run multiple releases for an extended
    period of time, or even continuously, using multiple release tracks.
    The tracks would be differentiated by labels.

###Services:

Kubernetes Pods are mortal. They are not resurrected. If some set of Pods
(let's call them backends) provides functionality to other Pods (let's call
them frontends) inside the Kubernetes cluster, how do those frontends find out
and keep track of which backends are in that set?

Service is an abstraction which defines a logical set of Pods and a policy by
which to access them - sometimes called a micro-service. The set of Pods
targeted by a Service is determined by a Label Selector.

As an example, consider an image-processing backend which is running with
3 replicas. Those replicas are mutually interchangeable - frontends do not care
which backend they use. While the actual Pods that compose the backend set may
change, the frontend clients should not need to manage that themselves. The
Service abstraction enables this decoupling.

**In addition to providing abstractions to access Pods, can also abstract**
**any kind of backend:**

  * you want to have an external database cluster in production, but in test
    you use your own databases.
  * you want to point your service to a service in another Namespace or on
    another cluster.
  * you are migrating your workload to Kubernetes and some of your backends
    run outside of Kubernetes.

###Service Proxies:

Every node in a Kubernetes cluster runs a kube-proxy. This application watches
the Kubernetes master for the addition and removal of Service and Endpoints
objects. For each Service it opens a port (random) on the local node. Any
connections made to that port will be proxied to one of the corresponding
backend Pods. Which backend to use is decided based on the AffinityPolicy of
the Service. Lastly, it installs iptables rules which capture traffic to the
Service's Port on the Service's portal IP and redirects that traffic to the
previously described port.

The net result is that any traffic bound for the Service is proxied to an
appropriate backend without the clients knowing anything about Kubernetes
or Services or Pods.

**2 primary modes of finding a Service:**

  * Environment Variables:
    When a Pod is run on a Node, the kubelet adds a set of environment
    variables for each active Service.
    **Ordering Requirement**:  any Service that a Pod wants to access must be
    created before the Pod itself, or else the environment variables will not
    be populated. 
  * DNS: Add-on DNS server that watches the Kubernetes API for new Services and
    creates a set of DNS records for each. If DNS has been enabled throughout
    the cluster then all Pods should be able to do name resolution of Services
    automatically.

####Portals:
Unlike Pod IP addresses, which actually route to a fixed
destination, Service IPs are not actually answered by a single host.
Instead, `iptables` are used to define "virtual" IP addresses which are
transparently redirected as needed. **The tuple of the Service IP and**
**the Service port is called the portal.**