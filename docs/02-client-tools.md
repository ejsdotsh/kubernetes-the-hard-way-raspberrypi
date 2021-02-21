# Installing the Client Tools

In this lab you will install the command line utilities required to complete this tutorial: [cfssl](https://github.com/cloudflare/cfssl), [cfssljson](https://github.com/cloudflare/cfssl), and [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl).

## Install CFSSL

The `cfssl` and `cfssljson` command line utilities will be used to provision a [PKI Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) and generate TLS certificates.

Download and install `cfssl` and `cfssljson`:

### OS X

```txt
$ curl -o cfssl https://github.com/cloudflare/cfssl/releases/download/v1.5.0/cfssl_1.5.0_darwin_amd64
$ curl -o cfssljson https://github.com/cloudflare/cfssl/releases/download/v1.5.0/cfssljson_1.5.0_darwin_amd64
$ chmod +x cfssl cfssljson
$ sudo mv cfssl cfssljson /usr/local/bin/
$
```

Some OS X users may experience problems using the pre-built binaries in which case [Homebrew](https://brew.sh) might be a better option:

```txt
brew install cfssl
```

### Linux

```txt
$ wget -q --show-progress --https-only --timestamping https://github.com/cloudflare/cfssl/releases/download/v1.5.0/cfssl_1.5.0_linux_amd64 -O cfssl && \
  wget -q --show-progress --https-only --timestamping https://github.com/cloudflare/cfssl/releases/download/v1.5.0/cfssljson_1.5.0_linux_amd64 -O cfssljson
cfssl                       100%[=========================================>]  14.41M  27.9MB/s    in 0.5s
cfssljson                   100%[=========================================>]   9.22M  26.4MB/s    in 0.3s

$ chmod +x cfssl cfssljson
$ sudo mv cfssl cfssljson /usr/local/bin/
```

Alternatively, if you have a working Go 1.12+ development environment:

```txt
$ go get -u -v github.com/cloudflare/cfssl/cmd/cfssl && \
  go get -u -v github.com/cloudflare/cfssl/cmd/cfssljson
```

### Verification

Verify that `cfssl` and `cfssljson` version 1.4.1 or higher is installed:
*I installed using `go get` on wsl2, your output might vary*

```txt
$ cfssl version
Version: dev
Runtime: go1.13.8
```

```txt
$ cfssljson -version
Version: dev
Runtime: go1.13.8
```

## Install kubectl

The `kubectl` command line utility is used to interact with the Kubernetes API Server. If you have Docker installed, you probably already have `kubectl`. If not, download and install `kubectl` from the official release binaries:

- OS X

```txt
$ curl -LO https://dl.k8s.io/release/v1.20.0/bin/darwin/amd64/kubectl
$ chmod +x kubectl
$ sudo mv kubectl /usr/local/bin/
$
```

- Linux

```txt
$ curl -LO https://dl.k8s.io/release/v1.20.0/bin/linux/amd64/kubectl
$ chmod +x kubectl
$ sudo mv kubectl /usr/local/bin/
$
```

- Verification

Verify `kubectl` version 1.19 or higher is installed:

(this is on wsl2 w/ docker)

```txt
$ kubectl version --client
Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.3", GitCommit:"1e11e4a2108024935ecfcb2912226cedeafd99df", GitTreeState:"clean", BuildDate:"2020-10-14T12:50:19Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"linux/amd64"}
```

Next: [Provisioning Compute Resources](03-compute-resources.md)
