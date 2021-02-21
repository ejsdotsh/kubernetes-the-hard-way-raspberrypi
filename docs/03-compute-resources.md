# Provisioning Compute Resources

Kubernetes requires a set of machines to host both the Kubernetes control plane and the worker nodes where containers are ultimately run. Preparing the RaspberryPi for Kubernetes is beyond the scope of this tutorial. Please see [RaspberryPik3s](https://github.com/joshuaejs/raspberry-pik3s) to see what I used to complete this tutorial.

## Networking

The Kubernetes [networking model](https://kubernetes.io/docs/concepts/cluster-administration/networking/#kubernetes-model) assumes a flat network in which containers and nodes can communicate with each other. In cases where this is not desired [network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) can limit how groups of containers are allowed to communicate with each other and external network endpoints.

> Setting up network policies is out of scope for this tutorial.

### Network Infoformation

There are no GCP network resources to instantiate, which means that there is also no automagic subnet creation. For this tutorial, I am using the following subnets:

- `INTERNAL_IP` range: 192.168.50.193-200
- `load balancer VIP`: 192.168.50.192
- `Service Cluster network`: 10.240.0.0/24
- `clusterCIDR`: 10.200.0.0/16

- rpi01: 192.168.50.193
  - controller
- rpi02: 192.168.50.194
  - controller
- rpi03: 192.168.50.195
  - controller
- rpi04: 192.168.50.196
  - podCIDR: 10.200.4.0/24
- rpi05: 192.168.50.197
  - podCIDR: 10.200.5.0/24
- rpi06: 192.168.50.198
  - podCIDR: 10.200.6.0/24
- rpi07: 192.168.50.199
  - podCIDR: 10.200.7.0/24
- rpi08: 192.168.50.200
  - podCIDR: 10.200.8.0/24

## RaspberryPi

The RaspberryPi in this lab will be provisioned using [Ubuntu Server](https://www.ubuntu.com/server) 20.10, which has updated code versions and good support for the [containerd container runtime](https://github.com/containerd/containerd). They are also each provisioned with a fixed [RFC1918](https://tools.ietf.org/html/rfc1918) address to simplify the Kubernetes bootstrapping process. Please see [Raspberry Pik3s](https://github.com/joshuaejs/raspberry-pik3s) for details on the installation base setup.

### Kubernetes Controllers

In Kubernetes, [controllers](https://kubernetes.io/docs/concepts/architecture/controller/) are `control loops` that try to move the *current state* of the cluster closer to the *desired state*. Three RaspberryPi, rpi0[1-3], will be used as `controllers`, and will also perform load-balancing duties.

### Kubernetes Nodes

A Kubernetes [node](https://kubernetes.io/docs/concepts/architecture/nodes/) is managed by the control plane, and is where `Pods` run containers/workloads. My five remaining RaspberryPi, rpi0[4-8], will be the `nodes`. Each node requires a pod subnet allocation from the Kubernetes cluster CIDR range. The pod subnet allocation will be used to configure container networking in a later exercise.

> The Kubernetes cluster CIDR range is defined by the Controller Manager's `--cluster-cidr` flag. In this tutorial, the cluster CIDR range is `10.200.0.0/16`, which supports 256 /24 subnets (pods)

```txt
$for ii in {4..8}; do echo rpi0${i} has podCIDR 10.200.${ii}.0/24; done
rpi04 has podCIDR 10.200.4.0/24
rpi05 has podCIDR 10.200.5.0/24
rpi06 has podCIDR 10.200.6.0/24
rpi07 has podCIDR 10.200.7.0/24
rpi08 has podCIDR 10.200.8.0/24
```

## Configuring SSH Access

Ubuntu Server 20.10 has SSH enabled by default, and the user is created with a playbook.

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
