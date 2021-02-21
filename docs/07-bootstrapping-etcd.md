# Bootstrapping the etcd Cluster

Kubernetes components are stateless and store cluster state in [etcd](https://github.com/etcd-io/etcd). In this lab you will bootstrap a three node etcd cluster and configure it for high availability and secure remote access.

## Prerequisites

The commands in this lab must be run on each controller instance: `rpi01`, `rpi02`, and `rpi03`; I recommend using tmux. Login to each controller using the `ssh` command. Example:

```txt
ssh rpi01
```

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.

## Bootstrapping an etcd Cluster Member

### Download and Install the etcd Binaries

Download the official etcd release binaries from the [etcd](https://github.com/etcd-io/etcd) GitHub project:

```txt
wget -q --show-progress --https-only --timestamping "https://github.com/etcd-io/etcd/releases/download/v3.4.14/etcd-v3.4.14-linux-arm64.tar.gz"
```

Extract and install the `etcd` server and the `etcdctl` command line utility:

```txt
tar -zxf etcd-v3.4.14-linux-arm64.tar.gz
sudo mv etcd-v3.4.14-linux-arm64/etcd* /usr/local/bin/
```

### Configure the etcd Server

```txt
sudo mkdir -p /etc/etcd /var/lib/etcd
sudo chmod 700 /var/lib/etcd
sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
```

create `/etc/etcd/etcd.conf`

```txt
cat <<EOF | sudo tee /etc/etcd/etcd.conf
ETCD_UNSUPPORTED_ARCH=arm64
EOF
```

The `INTERNAL_IP` is the address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address for the current compute instance:

```txt
INTERNAL_IP=$(host $(hostname -s).octopik3s.io | awk '{print $4}')
```

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:

```txt
ETCD_NAME=$(hostname -s)
```

Create the `etcd.service` systemd unit file:

```txt
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
EnvironmentFile=/etc/etcd/etcd.conf
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster rpi01=https://192.168.50.193:2380,rpi02=https://192.168.50.194:2380,rpi03=https://192.168.50.195:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the etcd Server

```txt
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
```

> Remember to run the above commands on each controller: `rpi01`, `rpi02`, and `rpi03`.

## Verification

List the etcd cluster members:

```txt
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

> output

```txt
601fa20fbb620f8a, started, rpi02, https://192.168.50.194:2380, https://192.168.50.194:2379, false
729d5d0a6cd23e32, started, rpi01, https://192.168.50.193:2380, https://192.168.50.193:2379, false
c6b9c056ddc35df5, started, rpi03, https://192.168.50.195:2380, https://192.168.50.195:2379, false
```

Next: [Bootstrapping the Kubernetes Control Plane](08-bootstrapping-kubernetes-controllers.md)
