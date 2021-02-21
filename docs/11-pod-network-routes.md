# Provisioning Pod Network Routes

Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing [static routes](https://en.wikipedia.org/wiki/Static_routing) on each node.

In this lab you will add routes for each worker node that maps the node's Pod CIDR range to the node's internal IP address.

> There are [other ways](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-achieve-this) to implement the Kubernetes networking model.

## The Routing Table

In this section you will generate the configuration files required to configure static routes on each node. This assumes that all RaspberryPi are connected to the same layer-2 network, using the following IP addressing:

- VIP: 192.168.50.192
- clusterCIDR: 10.200.0.0/16
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

Each worker node needs routes to the other podCIDR subnets.

```yaml
HOSTNUM=`hostname -s | cut -c5`

cat > 50-cloud-init.yaml <<EOF
# Kubernetes on RaspberryPi The Hard Way
# worker network configuration
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
      match:
        driver: bcmgenet smsc95xx lan78xx
      optional: false
      set-name: eth0
      routes:
EOF
```

```sh
for pi in {4..8}
do
  if [ ${pi} -ne ${HOSTNUM} ]
  then
    echo "      - to: 10.200.${pi}.0/24" >> 50-cloud-init.yaml
    echo "        via: 192.168.50.${pi}" >> 50-cloud-init.yaml
    echo "        metric: 100" >> 50-cloud-init.yaml
  elif [ ${pi} -eq ${HOSTNUM} ]
  then
    echo -n
  fi
done
```

## Routes

### Controllers

Configure the routes on the controllers:

```yaml
cat <<EOF | sudo tee /etc/netplan/50-cloud-init.yaml
# Kubernetes on RaspberryPi The Hard Way
# controller network configuration
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
      match:
        driver: bcmgenet smsc95xx lan78xx
      optional: false
      set-name: eth0
      routes:
      - to: 10.200.4.0/24
        via: 192.168.50.4
        metric: 100
      - to: 10.200.5.0/24
        via: 192.168.50.5
        metric: 100
      - to: 10.200.6.0/24
        via: 192.168.50.6
        metric: 100
      - to: 10.200.7.0/24
        via: 192.168.50.7
        metric: 100
      - to: 10.200.8.0/24
        via: 192.168.50.8
        metric: 100
EOF
```

Apply the configuration:

`sudo netplan apply`

Verify the routes on each controller:

```txt
rpi02:~$ ip route
default via 192.168.50.1 dev eth0 proto dhcp src 192.168.50.194 metric 100
10.200.4.0/24 via 192.168.50.4 dev eth0 proto static metric 100 onlink
10.200.5.0/24 via 192.168.50.5 dev eth0 proto static metric 100 onlink
10.200.6.0/24 via 192.168.50.6 dev eth0 proto static metric 100 onlink
10.200.7.0/24 via 192.168.50.7 dev eth0 proto static metric 100 onlink
10.200.8.0/24 via 192.168.50.8 dev eth0 proto static metric 100 onlink
192.168.50.0/24 dev eth0 proto kernel scope link src 192.168.50.194
192.168.50.1 dev eth0 proto dhcp scope link src 192.168.50.194 metric 100
```

### Workers

Move the configuration file into place, and apply

```txt
sudo mv 50-cloud-init.yaml /etc/netplan/
sudo netplan apply
```

Verify the routes on each worker:

```txt
rpi06:~$ ip route
default via 192.168.50.1 dev eth0 proto dhcp src 192.168.50.198 metric 100
10.200.4.0/24 via 192.168.50.4 dev eth0 proto static metric 100 onlink
10.200.5.0/24 via 192.168.50.5 dev eth0 proto static metric 100 onlink
10.200.6.0/24 dev cnio0 proto kernel scope link src 10.200.6.1 linkdown
10.200.7.0/24 via 192.168.50.7 dev eth0 proto static metric 100 onlink
10.200.8.0/24 via 192.168.50.8 dev eth0 proto static metric 100 onlink
192.168.50.0/24 dev eth0 proto kernel scope link src 192.168.50.198
192.168.50.1 dev eth0 proto dhcp scope link src 192.168.50.198 metric 100
```

Next: [Deploying the DNS Cluster Add-on](12-dns-addon.md)
