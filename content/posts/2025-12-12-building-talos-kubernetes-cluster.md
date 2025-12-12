---
title: "Building a Talos Kubernetes Cluster with KubeSpan and Tailscale"
date: 2025-12-12
categories:
  - Howtos
  - Kubernetes
tags:
  - talos
  - kubernetes
  - tailscale
  - kubespan
  - homelab
draft: false
---

This guide walks through setting up a highly available Talos Linux Kubernetes cluster with 3 control plane nodes and 2 workers, featuring KubeSpan for encrypted node-to-node communication and Tailscale integration for secure remote access.

## Cluster Overview

| Node | Hostname | IP Address |
|------|----------|------------|
| Control Plane 1 | talos-cp01 | 172.16.18.231 |
| Control Plane 2 | talos-cp02 | 172.16.18.232 |
| Control Plane 3 | talos-cp03 | 172.16.18.233 |
| Worker 1 | talos-worker01 | 172.16.18.241 |
| Worker 2 | talos-worker02 | 172.16.18.242 |
| VIP Endpoint | - | 172.16.18.222 |

## Prerequisites

- Talos Linux installed on all nodes (using nocloud image)
- `talosctl` CLI installed on your workstation
- Tailscale account with an auth key
- Network connectivity to all nodes

## Environment Setup

Set up the environment variables for your cluster:

```bash
export CONTROL_PLANE_IP=("172.16.18.231" "172.16.18.232" "172.16.18.233")
export WORKER_IP=("172.16.18.241" "172.16.18.242")
export YOUR_ENDPOINT=172.16.18.222
export CLUSTER_NAME=talos-lon
export TALOSCONFIG=~/.talos/config
```

## Generate Cluster Secrets

First, generate the cluster secrets that will be used to create the configuration:

```bash
talosctl gen secrets -o secrets.yaml
```

Keep `secrets.yaml` safe - you'll need it to regenerate configs or add nodes later.

## Generate Cluster Configuration

Generate the Talos configuration files with KubeSpan enabled:

```bash
talosctl gen config --with-secrets secrets.yaml \
  --with-kubespan \
  $CLUSTER_NAME \
  https://$YOUR_ENDPOINT:6443 \
  --install-disk=/dev/vda \
  --install-image=factory.talos.dev/nocloud-installer/077514df2c1b6436460bc60faabc976687b16193b8a1290fda4366c69024fec2:v1.11.5
```

This generates:
- `controlplane.yaml` - Configuration for control plane nodes
- `worker.yaml` - Configuration for worker nodes
- `talosconfig` - Client configuration for `talosctl`

## Create Patch Files

Create a `patches` directory for node-specific configurations:

```bash
mkdir -p patches
```

### Tailscale Extension Patch

Create `patches/tailscale.yaml` to enable Tailscale on all nodes:

```yaml
apiVersion: v1alpha1
kind: ExtensionServiceConfig
name: tailscale
environment:
  - TS_AUTHKEY=tskey-auth-xxxxx  # Replace with your Tailscale auth key
```

### Hostname Patches

Create hostname patches for each node. These also configure DNS to use Tailscale's MagicDNS (100.100.100.100) for seamless name resolution across your tailnet.

**patches/cp01-hostname.yaml:**
```yaml
machine:
  network:
    hostname: talos-cp01
    nameservers:
      - 100.100.100.100
      - 1.1.1.1
      - 8.8.8.8
    searchDomains:
      - mule-company.ts.net
```

**patches/cp02-hostname.yaml:**
```yaml
machine:
  network:
    hostname: talos-cp02
    nameservers:
      - 100.100.100.100
      - 1.1.1.1
      - 8.8.8.8
    searchDomains:
      - mule-company.ts.net
```

**patches/cp03-hostname.yaml:**
```yaml
machine:
  network:
    hostname: talos-cp03
    nameservers:
      - 100.100.100.100
      - 1.1.1.1
      - 8.8.8.8
    searchDomains:
      - mule-company.ts.net
```

**patches/worker01-hostname.yaml:**
```yaml
machine:
  network:
    hostname: talos-worker01
    nameservers:
      - 100.100.100.100
      - 1.1.1.1
      - 8.8.8.8
    searchDomains:
      - mule-company.ts.net
```

**patches/worker02-hostname.yaml:**
```yaml
machine:
  network:
    hostname: talos-worker02
    nameservers:
      - 100.100.100.100
      - 1.1.1.1
      - 8.8.8.8
    searchDomains:
      - mule-company.ts.net
```

> **Note:** The `100.100.100.100` DNS server is Tailscale's MagicDNS resolver, which allows nodes to resolve names within your tailnet. The search domain should match your Tailscale network's domain.

## Verify Node Connectivity

Before applying configurations, verify that you can reach all nodes and check their network links:

```bash
# Check network links on all nodes
talosctl --nodes 172.16.18.231 get links --insecure
talosctl --nodes 172.16.18.232 get links --insecure
talosctl --nodes 172.16.18.233 get links --insecure
talosctl --nodes 172.16.18.241 get links --insecure
talosctl --nodes 172.16.18.242 get links --insecure
```

Verify the disks are available:

```bash
# Check available disks on all nodes
talosctl get disks --insecure --nodes 172.16.18.231
talosctl get disks --insecure --nodes 172.16.18.232
talosctl get disks --insecure --nodes 172.16.18.233
talosctl get disks --insecure --nodes 172.16.18.241
talosctl get disks --insecure --nodes 172.16.18.242
```

Ensure `/dev/vda` (or your target disk) is available on all nodes.

## Apply Configuration to Nodes

### Control Plane Nodes

Apply the configuration to each control plane node with their respective hostname patches:

```bash
talosctl apply-config --insecure --nodes 172.16.18.231 \
  --file controlplane.yaml \
  --config-patch @patches/cp01-hostname.yaml \
  --config-patch @patches/tailscale.yaml

talosctl apply-config --insecure --nodes 172.16.18.232 \
  --file controlplane.yaml \
  --config-patch @patches/cp02-hostname.yaml \
  --config-patch @patches/tailscale.yaml

talosctl apply-config --insecure --nodes 172.16.18.233 \
  --file controlplane.yaml \
  --config-patch @patches/cp03-hostname.yaml \
  --config-patch @patches/tailscale.yaml
```

### Worker Nodes

Apply the configuration to each worker node:

```bash
talosctl apply-config --insecure --nodes 172.16.18.241 \
  --file worker.yaml \
  --config-patch @patches/worker01-hostname.yaml \
  --config-patch @patches/tailscale.yaml

talosctl apply-config --insecure --nodes 172.16.18.242 \
  --file worker.yaml \
  --config-patch @patches/worker02-hostname.yaml \
  --config-patch @patches/tailscale.yaml
```

The nodes will reboot and install Talos Linux.

## Configure talosctl

Set up the `talosctl` client configuration:

```bash
mkdir -p ~/.talos
cp ./talosconfig ~/.talos/config
export TALOSCONFIG=~/.talos/config
```

Configure the endpoints for the control plane:

```bash
talosctl config endpoint 172.16.18.231 172.16.18.232 172.16.18.233
```

## Bootstrap the Cluster

Bootstrap etcd on the first control plane node. This only needs to be done once:

```bash
talosctl bootstrap --nodes 172.16.18.231
```

Wait for the bootstrap process to complete. You can monitor progress with:

```bash
talosctl --nodes 172.16.18.231 dmesg -f
```

## Retrieve Kubeconfig

Once the cluster is bootstrapped, retrieve the kubeconfig:

```bash
talosctl kubeconfig --nodes 172.16.18.231
```

This merges the cluster's kubeconfig into your default `~/.kube/config` file.

## Verify the Cluster

Check that all nodes have joined the cluster:

```bash
kubectl get nodes -o wide
```

You should see all 5 nodes in `Ready` state:

```
NAME             STATUS   ROLES           AGE   VERSION
talos-cp01       Ready    control-plane   5m    v1.32.x
talos-cp02       Ready    control-plane   5m    v1.32.x
talos-cp03       Ready    control-plane   5m    v1.32.x
talos-worker01   Ready    <none>          4m    v1.32.x
talos-worker02   Ready    <none>          4m    v1.32.x
```

## Verify KubeSpan

Check that KubeSpan is working correctly:

```bash
talosctl --nodes 172.16.18.231 get kubespanpeerstatus
```

## Verify Tailscale

Check that Tailscale is connected on your nodes:

```bash
talosctl --nodes 172.16.18.231 services tailscale
```

Your nodes should now appear in your Tailscale admin console and be accessible via their Tailscale IPs or MagicDNS names.

## Summary

You now have a production-ready Talos Kubernetes cluster with:

- **3 control plane nodes** for high availability
- **2 worker nodes** for running workloads
- **KubeSpan** for encrypted WireGuard-based node communication
- **Tailscale** for secure remote access and MagicDNS resolution

## Useful Commands

```bash
# Check cluster health
talosctl --nodes 172.16.18.231 health

# View cluster members
talosctl --nodes 172.16.18.231 get members

# Check etcd status
talosctl --nodes 172.16.18.231 etcd members

# Upgrade a node
talosctl --nodes <node-ip> upgrade --image <new-image>

# Reset a node (destructive!)
talosctl --nodes <node-ip> reset --graceful
```
