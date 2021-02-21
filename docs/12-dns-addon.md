# Deploying the DNS Cluster Add-on

In this lab you will deploy the [DNS add-on](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) which provides DNS based service discovery, backed by [CoreDNS](https://coredns.io/), to applications running inside the Kubernetes cluster.

## The DNS Cluster Add-on

Verify the [tag](https://hub.docker.com/r/coredns/coredns/tags); using `latest` is not recommended, so the latest version was chosen, and the deployment yaml updated.

Deploy the `coredns` cluster add-on:

```txt
kubectl apply -f ./ coredns-1.8.0.yaml
```

> output

```txt
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
```

List the pods created by the `kube-dns` deployment:

```txt
$ kubectl get pods -l k8s-app=kube-dns -n kube-system
NAME                       READY   STATUS        RESTARTS   AGE
coredns-6cff99dc8c-h5q55   1/1     Running       0          23s
coredns-6cff99dc8c-r2gh5   1/1     Running       0          23s
```

## Verification

Create a `busybox` deployment:

```txt
kubectl run busybox --image=busybox:1.28 --command -- sleep 3600
```

List the pod created by the `busybox` deployment:

```txt
kubectl get pods -l run=busybox
```

> output

```txt
NAME      READY   STATUS    RESTARTS   AGE
busybox   1/1     Running   0          3s
```

```txt
kubectl exec -ti busybox -- nslookup kubernetes
```

> output
  **TODO**

```txt
Server:    10.240.0.10
Address 1: 10.240.0.10

nslookup: can't resolve 'kubernetes'
command terminated with exit code 1
```

Next: [Smoke Test](13-smoke-test.md)
