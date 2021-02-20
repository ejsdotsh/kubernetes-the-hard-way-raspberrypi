# Kubernetes The Hard Way on RaspberryPi

This tutorial walks you through setting up Kubernetes the hard way, on RaspberryPi. It is a fork of the incomprable [@KelseyHightower's](https://github.com/kelseyhightower) [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way), heavily modified and/or rewritten to use RaspberyPi instead of GCP. This guide is not for people looking for a fully automated command to bring up a RaspberryPi Kubernetes cluster. If that's you, then check out [Rancher's K3s](https://rancher.com/docs/k3s/latest/en/), or the [Getting Started Guides](https://kubernetes.io/docs/setup).

Kubernetes The Hard Way on RaspberryPi is optimized for learning, which means taking the long route to ensure you understand each task required to bootstrap a RaspberryPi Kubernetes cluster.

> The results of this tutorial are even less production ready than the original, and may receive limited support from the community, but don't let that stop you from learning!

## Copyright

This work, like the original, is licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode).

This tutorial has been updated and modified to reflect the differences in availability and versions of pre-compiled binaries for the RaspberryPi's arm64 hardwre, versus GCP's amd64, as well as not having the GCP management plane to provision resources and control access. It is not associated with, nor endorsed by, Kelsey Hightower.

## Target Audience

The target audience for this tutorial is someone planning to support a production Kubernetes cluster and who wants to understand how everything fits together.

## Cluster Details

Kubernetes The Hard Way on RaspberryPi, guides you through bootstrapping a highly available RaspberryPi Kubernetes cluster with end-to-end encryption between components and RBAC authentication.

* [kubernetes](https://kubernetes.io/docs/setup/release/notes/) v1.20.0
* [containerd](https://github.com/containerd/containerd) (using apt) v1.3.7-0ubuntu3.2
* [coredns](https://github.com/coredns/coredns/releases) v1.8.1
* [cni](https://github.com/containernetworking/cni) v0.9.1
* [etcd](https://github.com/etcd-io/etcd/releases) v3.4.14
* [ubuntu server](https://ubuntu.com/download/raspberry-pi) v20.10

## Labs

This tutorial assumes you have access to at least four RasberryPi 4B computers.

* [Prerequisites](docs/01-prerequisites.md)
* [Installing the Client Tools](docs/02-client-tools.md)
* [Provisioning Compute Resources](docs/03-compute-resources.md)
* [Provisioning the CA and Generating TLS Certificates](docs/04-certificate-authority.md)
* [Generating Kubernetes Configuration Files for Authentication](docs/05-kubernetes-configuration-files.md)
* [Generating the Data Encryption Config and Key](docs/06-data-encryption-keys.md)
* [Bootstrapping the etcd Cluster](docs/07-bootstrapping-etcd.md)
* [Bootstrapping the Kubernetes Control Plane](docs/08-bootstrapping-kubernetes-controllers.md)
* [Bootstrapping the Kubernetes Worker Nodes](docs/09-bootstrapping-kubernetes-workers.md)
* [Configuring kubectl for Remote Access](docs/10-configuring-kubectl.md)
* [Provisioning Pod Network Routes](docs/11-pod-network-routes.md)
* [Deploying the DNS Cluster Add-on](docs/12-dns-addon.md)
* [Smoke Test](docs/13-smoke-test.md)
* [Cleaning Up](docs/14-cleanup.md)
