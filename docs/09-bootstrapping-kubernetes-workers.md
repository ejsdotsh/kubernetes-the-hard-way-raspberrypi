# Bootstrapping the Kubernetes Worker Nodes

In this lab you will bootstrap five Kubernetes nodes. The following components will be installed on each node: [runc](https://github.com/opencontainers/runc), [container networking plugins](https://github.com/containernetworking/cni), [containerd](https://github.com/containerd/containerd), [kubelet](https://kubernetes.io/docs/admin/kubelet), and [kube-proxy](https://kubernetes.io/docs/concepts/cluster-administration/proxies).

## Prerequisites

The commands in this lab must be run on each node: `rpi04`, `rpi05`, `rpi06`, `rpi07`, and `rpi08`. Using tmux is highly recommended.

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.

## Provisioning a Kubernetes Worker Node

Install the OS dependencies:

```txt
sudo apt-get update && sudo apt-get -y install socat conntrack ipset
```

> The socat binary enables support for the `kubectl port-forward` command.

### Disable Swap

By default the kubelet will fail to start if [swap](https://help.ubuntu.com/community/SwapFaq) is enabled. It is [recommended](https://github.com/kubernetes/kubernetes/issues/7294) that swap be disabled to ensure Kubernetes can provide proper resource allocation and quality of service.

Verify that swap is not enabled:
*the [raspberrypik3s](https://github.com/joshuaejs/raspberry-pik3s) installation ensures that swap is disabled*

```txt
sudo swapon --show
```

If output is empthy then swap is not enabled. If swap is enabled run the following command to disable swap immediately:

```txt
sudo swapoff -a
```

> To ensure swap remains off after reboot consult your Linux distro documentation.

### Download and Install Worker Binaries

- [cni-plugins](https://github.com/containernetworking/plugins/releases/)
  - ~~the `overlay` kernel module doesn't exist/won't load which using these binaries~~
  - ~~using the apt package `containernetworking-plugins`~~
- [runc](https://github.com/opencontainers/runc/releases)
  - there is no arm64 binary; will have to use apt
- [containerd](https://github.com/containerd/containerd/releases)
  - there is no arm64 binary, will have to use apt
- [kubernetes binaries](https://kubernetes.io/docs/setup/release/notes/)
- [cri-tools](https://github.com/kubernetes-sigs/cri-tools/releases/)

```txt
sudo apt-get update && sudo apt-get install -y runc containerd

wget -q --show-progress --https-only --timestamping \
  https://github.com/containernetworking/plugins/releases/download/v0.9.1/cni-plugins-linux-arm64-v0.9.1.tgz \
  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.20.0/crictl-v1.20.0-linux-arm64.tar.gz \
  https://dl.k8s.io/v1.20.0/kubernetes-node-linux-arm64.tar.gz
```

Create the installation directories:

```txt
sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```txt
sudo tar -zxf crictl-v1.20.0-linux-arm64.tar.gz -C /usr/local/bin/
sudo tar -zxf cni-plugins-linux-arm64-v0.9.1.tgz -C /opt/cni/bin/
tar -zxf kubernetes-node-linux-arm64.tar.gz
rm -f kubernetes/node/bin/kubeadm
sudo mv kubernetes/node/bin/k* /usr/local/bin/
rm -rf kubernetes
```

### Configure CNI Networking

Pod CIDR ranges:

```txt
rpi04 has podCIDR 10.200.4.0/24
rpi05 has podCIDR 10.200.5.0/24
rpi06 has podCIDR 10.200.6.0/24
rpi07 has podCIDR 10.200.7.0/24
rpi08 has podCIDR 10.200.8.0/24
```

Create the `bridge` network configuration file:

```txt
POD_CIDR=10.200.`(hostname -s | cut -c5)`.0/24

cat <<EOF | sudo tee /etc/cni/net.d/10-bridge.conf
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "${POD_CIDR}"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF
```

Create the `loopback` network configuration file:

```txt
cat <<EOF | sudo tee /etc/cni/net.d/99-loopback.conf
{
    "cniVersion": "0.3.1",
    "name": "lo",
    "type": "loopback"
}
EOF
```

### Configure containerd

Some prerequisites as documented on [container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)

```txt
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

Install the arm64 binaries

```txt
sudo apt-get update && sudo apt-get install -y containerd runc
```

Create the `containerd` configuration file, using the default configuration as a starting point:

```txt
sudo mkdir -p /etc/containerd/
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

Enable systemd to use cgroups

```txt
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
  ...
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
        ...
        SystemdCgroup = true
EOF
```

Reboot the node.

The systemd `containerd.service` is created when installed by apt.

- references
  - [container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)
  - [migrating k8s from docker to containerd](https://josebiro.medium.com/migrating-k8s-from-docker-to-containerd-484aaf6baf40)

### Configure the Kubelet

```txt
sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
sudo mv ca.pem /var/lib/kubernetes/
```

Create the `kubelet-config.yaml` configuration file:

```txt
POD_CIDR=10.200.`(hostname -s | cut -c5)`.0/24

cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.240.0.10"
podCIDR: "${POD_CIDR}"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF
```

> The `resolvConf` configuration is used to avoid loops when using CoreDNS for service discovery on systems running `systemd-resolved`.

Create the `kubelet.service` systemd unit file:

```txt
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --cni-bin-dir="/opt/cni/bin" \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Proxy

```txt
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```txt
cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
masqueradeAll: "true"
clusterCIDR: "10.200.0.0/16"
EOF
```

Create the `kube-proxy.service` systemd unit file:

```txt
cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Worker Services

```txt
sudo systemctl daemon-reload
sudo systemctl enable kubelet kube-proxy
sudo systemctl start kubelet kube-proxy
```

> Remember to run the above commands on each worker node: `rpi04`, `rpi05`, `rpi06`, `rpi07`, and `rpi08`.

## Verification

At this point, everything should be running on the nodes, and they should have registered with the controllers. List the registered Kubernetes nodes:

```txt
$ ssh rpi01 "kubectl get nodes --kubeconfig admin.kubeconfig"
NAME    STATUS   ROLES    AGE     VERSION
rpi04   Ready    <none>   3m36s   v1.20.0
rpi05   Ready    <none>   3m40s   v1.20.0
rpi06   Ready    <none>   3m36s   v1.20.0
rpi07   Ready    <none>   3m36s   v1.20.0
rpi08   Ready    <none>   3m38s   v1.20.0
```

Next: [Configuring kubectl for Remote Access](10-configuring-kubectl.md)
