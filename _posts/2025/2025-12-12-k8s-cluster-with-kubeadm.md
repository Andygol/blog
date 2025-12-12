---
layout: "post"
title: "Deploying a Kubernetes Cluster on a Local Machine: A Complete Step-by-Step Guide"
date: "2025-12-12 08:30:00 +0200"
categories: en
tags:
  - Kubernetes
  - Containers
  - Orchestration
  - DevOps
  - Infrastructure
  - kubeadm
  - flannel
  - containerd
comments: true
lang: en
ref: 2025-12-12-k8s-cluster-with-kubeadm
---

> 💡 You won't find such a detailed step-by-step guide in the official Kubernetes documentation! The official docs contain separate recommendations. Here everything is gathered in one place so you can quickly and easily create your first cluster.

## What we'll do

We'll create a full Kubernetes cluster on your local machine. If you have separate physical machines, you can adapt this guide to run on physical hardware. For this guide we will use:

- **Multipass** — tool to create Ubuntu virtual machines
- **kubeadm** — primary tool to initialize the cluster
- **Flannel** — CNI plugin to create Pod networking
- **MetalLB** — load balancer for our cluster

Our cluster will consist of **three nodes**:

- 1 control plane node
- 2 worker nodes

<iframe width="560" height="315" src="https://www.youtube.com/embed/ji3nKGN16hQ?si=ZHCl9hJyLgmTPLCn" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Prerequisites

Before starting, install [Multipass](https://canonical.com/multipass):

```bash
brew install multipass
```

## Step 1: Create virtual machines

### Define the list of nodes

Create an array with the names of the three virtual machines. This is just a simple list of the machines we need.

```bash
NODES=(k8s-control k8s-worker-1 k8s-worker-2)
```

### Create VMs

Now use multipass to create three Ubuntu virtual machines.

```bash
for NODE in "${NODES[@]}"; do
  multipass launch --name $NODE --cpus 2 --memory 4G --disk 20G
done
```

Using the `for NODE in "${NODES[@]}"` loop we iterate over each name; `multipass launch --name $NODE` creates a VM with the given name and these parameters:

- `--cpus 2` — allocate 2 CPU cores (minimum for K8s)
- `--memory 4G` — allocate 4 GB of RAM
- `--disk 20G` — allocate 20 GB of disk space

Multipass will automatically download Ubuntu. This will take a few minutes ☕.

## Step 2: Prepare all nodes

Once the virtual machines are created, start configuring them. These steps need to be executed on all three machines.

### 2.1. System update

```bash
echo "=== [1/7] Updating system on all nodes ==="
for NODE in "${NODES[@]}"; do
  multipass exec $NODE -- bash -c "
    sudo apt-get update &&
    sudo apt-get upgrade -y
  "
done
```

- `multipass exec $NODE` — runs a command on the VM
- `sudo apt-get update` — updates the package lists
- `sudo apt-get upgrade -y` — installs all upgrades; `-y` answers yes to prompts

It's important to start with an up-to-date system with the latest security fixes and package updates.

After this command all packages will be updated to the latest versions.

### 2.2. Disable firewall

```bash
echo "=== [2/7] Disabling firewall on all nodes ==="
for NODE in "${NODES[@]}"; do
  multipass exec $NODE -- sudo ufw disable
done
```

UFW is Ubuntu's firewall. For a learning cluster we disable the firewall to avoid networking issues between nodes.

⚠️ **In production** you should configure the firewall properly and open required ports!

### 2.3. Load kernel modules

```bash
echo "=== [3/7] Configuring kernel modules ==="
for NODE in "${NODES[@]}"; do
  multipass exec $NODE -- bash -c "
    echo -e 'overlay\nbr_netfilter' | sudo tee /etc/modules-load.d/k8s.conf
    sudo modprobe overlay
    sudo modprobe br_netfilter
  "
done
```

Kubernetes requires these two Linux kernel modules to be enabled:

- `overlay` — for container filesystem layering
- `br_netfilter` — for network connectivity between containers

The first line writes these modules to a config file so they're loaded at boot. The next two lines load them now.

### 2.4. Configure sysctl networking parameters

```bash
echo "=== [4/7] Configuring networking sysctl parameters ==="
for NODE in "${NODES[@]}"; do
  multipass exec $NODE -- bash -c "
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
    sudo sysctl --system
  "
done
```

We configure kernel networking settings:

- bridge-nf-call-iptables — allows iptables to see bridged traffic (IPv4 and IPv6)
- ip_forward — enables packet forwarding between interfaces

`sysctl --system` applies these settings immediately.

This is critical for Kubernetes networking!

### 2.5. Install containerd

```bash
echo "=== [5/7] Installing containerd ==="
for NODE in "${NODES[@]}"; do
  multipass exec $NODE -- sudo apt-get install -y containerd
done
```

Containerd is the runtime that runs and manages containers. Kubernetes supports multiple runtimes; containerd is a recommended and popular choice.

### 2.6. Configure containerd

```bash
echo "=== [6/7] Configuring containerd and CRI ==="
for NODE in "${NODES[@]}"; do
  multipass exec $NODE -- bash -c "
    sudo mkdir -p /etc/containerd
    containerd config default | sudo tee /etc/containerd/config.toml

    # Update the sandbox image
    sudo sed -i 's/registry.k8s.io\\/pause:3.8/registry.k8s.io\\/pause:3.10.1/' /etc/containerd/config.toml

    # Enable cgroup via systemd
    sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

    # Add crictl config
    sudo tee /etc/crictl.yaml <<EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF

    sudo systemctl restart containerd
    sudo systemctl enable containerd
  "
done
```

**What happens here:**

1. Generate the default containerd config into /etc/containerd/config.toml
2. Update the Kubernetes pause image to version 3.10.1
3. Enable SystemdCgroup to use systemd for cgroup management
4. Configure crictl to talk to containerd
5. Restart and enable containerd

### 2.7. Install Kubernetes components

```bash
echo "=== [7/7] Installing Kubernetes components ==="
for NODE in "${NODES[@]}"; do
  multipass exec $NODE -- bash -c "
    sudo apt-get install -y apt-transport-https ca-certificates curl gpg

    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key \
      | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' \
      | sudo tee /etc/apt/sources.list.d/kubernetes.list

    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
    sudo systemctl enable kubelet
  "
done
```

1. Prepare tools for secure package installation (HTTPS, certificates, GPG)
2. Add the official Kubernetes signing key
3. Add the Kubernetes v1.34 repository
4. Install:
   - `kubelet` — agent on each node
   - `kubeadm` — initializer tool
   - `kubectl` — CLI client

`apt-mark hold` prevents automatic updates; component versions must match across the cluster. Enable kubelet.

### 2.8. Done! ✅

These steps can be combined into a single script to prepare all nodes for cluster creation.

All three machines are now ready to become part of a Kubernetes cluster.[^1]

## Step 3: Initialize the control plane

### Connect to the control plane

```bash
multipass shell k8s-control
```

This opens a shell inside the `k8s-control` VM. Now we're working directly on that machine.

### Get the control plane IP

```bash
CONTROL_IP=$(hostname -I | awk '{print $1}')
```

- `hostname -I` — shows all IP addresses of the machine
- `awk '{print $1}'` — extracts the first address
- `$()` — stores the result in CONTROL_IP

We need the IP so worker nodes know where to connect.

### Initialize the cluster 🚀

```bash
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=$CONTROL_IP
```

kubeadm init initializes the control plane (API server, scheduler, controller-manager). At the end it prints a `kubeadm join` command to run on worker nodes.

**Parameters:**

- `--pod-network-cidr=10.244.0.0/16` — Pod network range (for Flannel)
- `--apiserver-advertise-address` — IP the API server advertises

⏱️ This takes 1–2 minutes.

**📝 Save the `kubeadm join` command output — you'll need it for workers!**

### Configure kubectl

kubeadm also prints instructions to configure kubectl for the current user:

```bash
mkdir -p ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
```

kubectl requires a configuration file to connect to the cluster.

1. `mkdir -p ~/.kube` — create kube config directory
2. `cp admin.conf ~/.kube/config` — copy admin kubeconfig
3. `chown` — change ownership so kubectl can run without sudo

## Step 4: Install CNI plugin — Flannel

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

**[Flannel](https://github.com/flannel-io/flannel#deploying-flannel-manually)** is a Container Network Interface (CNI) plugin that enables Pod networking across nodes.

Kubernetes does not provide cluster networking by itself; Flannel deploys the necessary DaemonSet, ConfigMap, ServiceAccount, and other resources.

## Step 5: Join worker nodes

During `kubeadm init` you received a `kubeadm join` command. Run it on each worker node.

```bash
for NODE in k8s-worker-1 k8s-worker-2; do
  multipass exec $NODE -- sudo kubeadm join 192.168.2.26:6443 \
    --token bsw6fd.e7624wl2688fybjx \
    --discovery-token-ca-cert-hash sha256:7850aa1c6181277e284a08b81256979db25698a89982f0885540376a5376e0bd
done
```

> ⚠️ **IMPORTANT:** In your case the IP, token and hash will be **different**! Use the command that `kubeadm init` printed.

What this does:

- `multipass exec $NODE --` — runs the join command on each worker
- `192.168.2.14:6443` — API server address (port 6443)
- `--token` — temporary token generated by kubeadm
- `--discovery-token-ca-cert-hash` — CA cert SHA256 hash to verify authenticity

## Step 6: Verify the cluster

Return to the control plane node and check node status.

```bash
multipass shell k8s-control
kubectl get nodes
```

You should see three nodes with STATUS `Ready`:

```text
NAME            STATUS   ROLES           AGE   VERSION
k8s-control     Ready    control-plane   5m    v1.34.0
k8s-worker-1    Ready    <none>          2m    v1.34.0
k8s-worker-2    Ready    <none>          2m    v1.34.0
```

If a node shows `NotReady`, wait a minute — Flannel may still be provisioning ⏳

## Step 7: Install MetalLB

### What is MetalLB?

**[MetalLB](https://metallb.io)** is a load balancer implementation for bare-metal clusters. In cloud-managed Kubernetes, LoadBalancer services are provided by the cloud provider. For a local cluster, MetalLB provides similar functionality.

### Apply manifests

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.15.3/config/manifests/metallb-native.yaml
```

### Wait for readiness

```bash
kubectl wait --namespace metallb-system \
  --for=condition=ready pod \
  --selector=app=metallb
```

This waits until all MetalLB pods are ready.

### Label worker nodes

To label all nodes whose names start with `k8s-worker-`, get node names and apply a label to each.

```bash
NODES=$(kubectl get nodes -o jsonpath='{.items[*].metadata.name}' | tr -s '[[:space:]]' '\n' | grep '^k8s-worker-')

for NODE in $NODES; do
  echo "Applying label to node: $NODE"
  kubectl label node "$NODE" metallb-role=worker --overwrite
done
```

We add the label `metallb-role=worker` to worker nodes. Labels help select resources in Kubernetes. MetalLB will run only on nodes with this label.

### Configure IP pool and L2Advertisement

```bash
CONTROL_IP=$(hostname -I | awk '{print $1}')
BASE_IP=$(echo $CONTROL_IP | cut -d. -f1-3)

cat <<EOF | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - ${BASE_IP}.200-${BASE_IP}.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: worker-nodes-l2
  namespace: metallb-system
spec:
  ipAddressPools:
  - first-pool
  nodeSelectors:
  - matchLabels:
      metallb-role: worker
EOF
```

**What this does:**

1. Get the base IP from the control plane IP (e.g. if control IP is `192.168.2.14`, BASE_IP becomes `192.168.2`)
2. `IPAddressPool` defines a range of external IPs MetalLB can allocate (.200–.250 in this example)
3. `L2Advertisement` configures MetalLB to announce those IPs via ARP (Layer 2)
4. `nodeSelectors` restrict MetalLB to nodes labeled `metallb-role=worker`

## Step 8: Demo 🎉

### Create a Deployment

```bash
kubectl create deployment hello \
  --image=nginxdemos/hello:plain-text \
  --replicas=3 \
  --port=80
```

Creates 3 replicas of a simple nginx demo app.

- `deployment` — manages a set of identical Pods
- `--image` — Docker image for the containers
- `--replicas=3` — run 3 copies
- `--port=80` — container port

### Create a LoadBalancer Service

```bash
kubectl expose deployment hello \
  --type=LoadBalancer \
  --port=80
```

A LoadBalancer service will get an external IP from MetalLB.

- `expose deployment` — creates a Service for the Deployment
- `--type=LoadBalancer` — request an external IP (provided by MetalLB)
- `--port=80` — service port

### Test load balancing

```bash
EXTERNAL_IP=$(kubectl get svc hello -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "LB IP: $EXTERNAL_IP"

for i in {1..10}; do
  echo "Request $i:"
  curl -s http://$EXTERNAL_IP | grep "Server address"
  echo "---"
done
```

This obtains the external IP allocated by MetalLB and makes 10 requests, showing which Pod served each request.

Expected result: requests are distributed across the three Pods.

```
Request 1:
Server address: 10.244.1.2:80
---
Request 2:
Server address: 10.244.2.3:80
---
Request 3:
Server address: 10.244.1.4:80
---
```

## 🚨 Important notes about using SWAP

By default Multipass VMs have swap disabled.

kubelet refuses to run if swap is enabled because swapping can cause unpredictable Pod behavior (containers being pushed to disk, slowing them down).

In our case swap is disabled by default — so it's fine! ✅

If you deploy Kubernetes on systems where swap is enabled:

**Option 1:** Turn off swap

```bash
sudo swapoff -a
# And comment out swap in /etc/fstab
```

**Option 2:** Configure kubelet to run with [swap support](https://andygol-k8s.netlify.app/docs/concepts/cluster-administration/swap-memory-management/) (experimental in K8s 1.28+)

## Useful commands

### Status checks

```bash
kubectl get nodes
kubectl get pods -A
kubectl get svc
```

### View logs

```bash
kubectl logs -n kube-system -l app=flannel
kubectl logs -n metallb-system -l app=metallb
```

### Delete resources

```bash
kubectl delete svc hello
kubectl delete deployment hello
```

### Stop the cluster

```bash
# Stop all VMs
for NODE in "${NODES[@]}"; do
  multipass stop $NODE
done

# Start again
for NODE in "${NODES[@]}"; do
  multipass start $NODE
done
```

### Remove the cluster

```bash
for NODE in "${NODES[@]}"; do
  multipass delete $NODE
done
multipass purge
```

## Summary

Congratulations! 🎉 You just created a full Kubernetes cluster on your local machine.

What we did:

- ✅ Created 3 virtual machines
- ✅ Configured containerd and kernel modules
- ✅ Initialized the control plane
- ✅ Joined worker nodes
- ✅ Installed Flannel for networking
- ✅ Configured MetalLB for LoadBalancer services
- ✅ Deployed a test app with load balancing

Now you have a local environment for:

- experimenting with Kubernetes
- testing manifests
- learning cluster architecture
- preparing for certifications (CKA, CKAD)

Hope this guide was useful!

If you have questions or issues — leave a comment. Share this post with colleagues who may find it helpful 🚀

Happy Kubernetes! ☸️

---

## Disclaimer 🚨

This cluster is intended for development and testing.

**For production use you need additional configuration:**

- Firewall and Network Policies
- RBAC (Role-Based Access Control)
- Secrets management
- Monitoring and logging
- etcd backups
- High Availability control plane
- Vulnerability scanning

---

[^1]: Preparation script <https://gist.github.com/Andygol/37d1397423e535bd0f7fabb593e81c41>
