# Configuring kubectl for Remote Access

In this lab you will generate a kubeconfig file for the `kubectl` command line utility based on the `admin` user credentials.

> Run the commands in this lab from the same directory used to generate the admin client certificates.

## The Admin Kubernetes Configuration File

Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability the IP address assigned to the external load balancer fronting the Kubernetes API Servers will be used.

Generate a kubeconfig file suitable for authenticating as the `admin` user:

```txt
VIP=192.168.50.192

kubectl config set-cluster kubernetes-on-raspberrpi-the-hard-way \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://${VIP}:8443

kubectl config set-credentials admin \
  --client-certificate=admin.pem \
  --client-key=admin-key.pem

kubectl config set-context kubernetes-on-raspberrpi-the-hard-way \
  --cluster=kubernetes-on-raspberrpi-the-hard-way \
  --user=admin

kubectl config use-context kubernetes-on-raspberrpi-the-hard-way
```

## Verification

Check the health of the remote Kubernetes cluster:
  **TODO: update ComponentStatus to non-deprecated version**

```txt
$ kubectl get componentstatus
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health":"true"}
etcd-2               Healthy   {"health":"true"}
etcd-1               Healthy   {"health":"true"}
```

List the nodes in the remote Kubernetes cluster:

```txt$ kubectl get nodes
NAME    STATUS   ROLES    AGE     VERSION
rpi04   Ready    <none>   9m24s   v1.20.0
rpi05   Ready    <none>   9m28s   v1.20.0
rpi06   Ready    <none>   9m24s   v1.20.0
rpi07   Ready    <none>   9m24s   v1.20.0
rpi08   Ready    <none>   9m26s   v1.20.0
```

Next: [Provisioning Pod Network Routes](11-pod-network-routes.md)
