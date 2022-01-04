+++
categories = ["kubernetes", "docker", "homelab"]
date = 2020-12-04T18:45:04Z
description = ""
draft = false
cover = "images/2020/12/teng-yuhong-qMehmIyaXvY-unsplash.jpg"
slug = "from-docker-to-containerd"
tags = ["kubernetes", "docker", "homelab"]
title = "Migrating Kubernetes from Docker to containerd"

+++


On December 2nd, a [surprise announcement](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/) made waves in the Kubernetes Twitter-sphere - that after the upcoming 1.20 release, Docker would be officially deprecated.

### Oh no!
Due to widespread confusion over what "Docker" means in specific contexts, many people panicked - myself included.  Because of its sheer popularity, Docker has become synonymous with "containers".  However, Docker is really an entire ecosystem of container tools and processes, including building and shipping container images.  The only thing Kubernetes is deprecating is using Docker as a container runtime, and the reasoning is sound.

Docker's lack of support for the "Container Runtime Interface" API - or CRI, for short - forced Kubernetes to implement an abstraction layer called "dockershim" to allow Kubernetes to manage containers in Docker.  The burden of maintaining dockershim was too great to bear, so they are deprecating dockershim in release 1.20, and will eventually remove it entirely in 1.22.

There are two other container runtimes featured in the Kubernetes quickstart guide as an alternative to Docker - `containerd` and CRI-O.  `containerd` is  the same runtime that Docker itself uses internally, just without the fancy Docker wrapping paper and tools.

### Ugh.
Annoyingly enough, I had recently finished migrating my entire homelab container infrastructure to Kubernetes three months ago, with Docker as the container runtime.  I initially thought, "Crap.  Guess I'll be rebuilding my cluster!"  Then I began to think about what such a change would look like, and whether replacing Docker with `containerd` in the same cluster is doable.

### Hmm...
Turns out, it is!

I have a 3-node HA cluster which I created using kubeadm.  Because I have multiple control plane nodes, I can remove them one at a time using `kubeadm reset`, rebuild them with `containerd` instead of Docker, and then rejoin using `kubeadm join`.

Here are the steps I came up with:

### Uninstalling Docker
1. Using `kubectl`, drain and evict pods from the target node.
```bash
kubectl drain ${node}
```

2. On the target node, use `kubeadm` to remove the node from the cluster.
```bash
kubeadm reset
```

3. Once `kubeadm reset` is finished, stop Docker and finish cleaning up the node.
```bash
systemctl stop docker
rm -rf /etc/cni/net.d
iptables --flush
```

4. Uninstall the Docker CE suite and CLI.
```bash
apt-get -y remove docker-ce*
rm -rf /var/lib/docker/*
rm -rf /var/lib/dockershim
```

5. Now's a great time to update your kernel and OS packages...
```bash
apt-get update
apt-get -y dist-upgrade
```

6. ...and reboot!
```bash
shutdown -r now
```

### Installing containerd
(These steps are lifted straight from the fantastic [k8s containerd docs](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)!)

7. Apply the module configs for `containerd`'s required kernel modules.
```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
```

8. Set sysctl tuning parameters for Kubernetes CRI
```bash
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sysctl --system
```

9. Install `containerd` if not already installed
```bash
apt-get install -y containerd.io
```

---
##### A note on filesystems
Since these nodes were running Docker, all of the container data is stored in /var/lib/docker.  With `containerd`, container data is now stored in /var/lib/containerd.  If you had the Docker data directory on its own filesystem, you'll need to remove it and create one for `containerd`.  The exact steps depend in your system, so I won't include them here.

##### Now back to the fun!
---

10. Generate a default configuration:
```bash
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
```

11. Modify the `config.toml` file generated above to enable the systemd cgroup driver:
```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

12. Now enable and start `containerd`!
```bash
systemctl enable --now containerd
```

13. Make sure `containerd` is happy before proceeding.
```bash
systemctl status containerd
```

Your system is now fully configured with the `containerd` runtime - but before we rejoin the cluster, there's one more step to get kubeadm to play nicely with it!

### Updating the kubelet configuration
Since this cluster was originally built with the Docker runtime, the default kubelet configuration does not explicitly set a cgroup driver.  By default, kubeadm with Docker auto-detects the cgroup driver - but other runtimes like `containerd` don't support that yet.  As a result, when you `kubeadm join` a `containerd` node without a cgroup driver specified, the kubelet won't start.  You can ninja-edit the `/var/lib/kubelet/config.yaml` file when joining and then restart the kubelet, but that's tedious and unnecessary.

Fortunately, we can update the baseline kubelet config at the cluster level to specify the right cgroup driver to use.

14. Edit the baseline kubelet config for your Kubernetes version - 1.18, 1.19, etc.
```bash
kubectl edit cm -n kube-system kubelet-config-1.18
```

15. Add the following entry for `cgroupDriver`:
```yaml
data:
  kubelet: |
  ...
    cgroupDriver: systemd
  ...
```

### Joining the cluster
16. Proceed to `kubeadm join` your node with the appropriate kubeadm command!  You can run `kubeadm token create --print-join-command` to create a new token.

```bash
kubeadm join 123.45.67.89:6443 \
  --token <...snip...> \
  --discovery-token-ca-cert-hash sha256:<...snip...> \
  [--control-plane --certificate-key <...snip...>]
```

For control plane nodes, be sure to include the `--control-plane` flag and `--certificate-key` for your cluster - otherwise the node will join as a worker!  I made this mistake and had to re-reset and rejoin the first node I converted.  Use `kubeadm init phase upload-certs --upload-certs` on another control plane node to reupload your certificates to the cluster, and then pass the provided certificate key to `kubeadm join`.

### Clean-up
Once your new node is joined, wait a few minutes for your CNI plugin to reprovision the networking stack.  Once you're satisfied and the node shows `Ready` in `kubectl get nodes`, you can uncordon the node with `kubectl uncordon`.

And finally, if necessary, don't forget to re-taint your new control plane node! When `kubeadm` rejoins the node, it applies the same default restriction to prevent control plane nodes from running worker pods.  I find separate control planes unnecessary for my homelab, so I taint them to allow pods to run anywhere.
```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

### Final thoughts
Now, granted - this process is *extremely* unnecessary, and runs contrary to the cloud ethos that nodes should be treated like cattle.  But for someone running a small bare-metal environment - where provisioning new nodes *isn't* entirely automated - these steps save a lot of time otherwise spent rebuilding VMs from the ground up, assigning IP addresses, updating DNS, and potentially building a whole new cluster.

And as an *added* bonus, I now know more about Kubernetes and container runtimes than I did last week.



