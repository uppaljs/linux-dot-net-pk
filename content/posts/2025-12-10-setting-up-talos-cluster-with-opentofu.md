---
title: "Setting Up a Talos Kubernetes Cluster on Proxmox Using OpenTofu"
date: 2025-12-10
categories:
  - Howtos
  - Kubernetes
tags:
  - talos
  - kubernetes
  - opentofu
  - terraform
  - proxmox
  - infrastructure-as-code
  - homelab
draft: false
---

## Introduction

[Talos Linux](https://www.talos.dev/) is a modern, minimal, and secure operating system designed specifically for running Kubernetes. Unlike traditional Linux distributions, Talos is immutable, API-driven, and removes SSH access entirely—making it ideal for production Kubernetes clusters.

In this article, I'll walk through how to provision a complete Talos Kubernetes cluster on Proxmox using [OpenTofu](https://opentofu.org/) (an open-source Terraform alternative). This approach leverages infrastructure as code to create reproducible, version-controlled cluster deployments.

## Architecture Overview

Our cluster will consist of:

- **3 Control Plane Nodes**: Running etcd and Kubernetes control plane components
- **2 Worker Nodes**: For running application workloads

All VMs will boot from the Talos ISO and use DHCP for network configuration with static leases configured on the DHCP server to ensure consistent IP addresses.

### Network Design

This setup relies on **DHCP with static leases**. Each VM is assigned a specific MAC address, and the DHCP server is pre-configured with static lease reservations that map these MAC addresses to fixed IP addresses. This approach provides:

- Consistent IP addressing without static configuration in VMs
- Centralized network management through the DHCP server
- Easy network changes without modifying VM configurations

## Prerequisites

Before starting, ensure you have:

1. **Proxmox VE** installed and configured
2. **OpenTofu** (or Terraform) installed locally
3. **Talos ISO** downloaded and uploaded to Proxmox storage
4. **DHCP server** with static lease support configured
5. **Proxmox API token** for authentication

### Downloading the Talos ISO from Image Factory

Instead of using the generic Talos ISO, we'll use the [Talos Image Factory](https://factory.talos.dev/) to generate a custom image with the system extensions we need. This approach bakes in the required extensions at boot time, which is cleaner than installing them post-boot.

Our custom image will include the following extensions:

- **iscsi-tools**: Required for iSCSI storage backends (Longhorn, OpenEBS)
- **qemu-guest-agent**: Enables Proxmox to communicate with the VM for proper shutdown/status
- **tailscale**: For secure remote access via Tailscale network
- **util-linux-tools**: Additional utilities for storage management

#### Using the Image Factory Web UI

1. Go to [https://factory.talos.dev/](https://factory.talos.dev/)
2. Select your Talos version (e.g., `v1.11.5`)
3. Choose platform: **nocloud** (for Proxmox/generic virtualization)
4. Select **System Extensions** and add:
   - `siderolabs/iscsi-tools`
   - `siderolabs/qemu-guest-agent`
   - `siderolabs/tailscale`
   - `siderolabs/util-linux-tools`
5. Download the ISO image

#### Using the Factory URL Directly

You can also construct the factory URL directly. The extensions are encoded in a schematic ID. For the extensions listed above, use:

```bash
# Download the custom Talos ISO with extensions
# This URL includes: iscsi-tools, qemu-guest-agent, tailscale, util-linux-tools
wget -O talos-v1.11.5-nocloud-amd64.iso \
  "https://factory.talos.dev/image/077514df2c1b6436460bc60faabc976687b16193b8a1290fda4366c69024fec2/v1.11.5/nocloud-amd64.iso"

# Upload to Proxmox storage
scp talos-v1.11.5-nocloud-amd64.iso root@proxmox-server:/var/lib/vz/template/iso/
```

> **Note:** The schematic ID (`077514df2c1b6436460bc60faabc976687b16193b8a1290fda4366c69024fec2`) encodes your specific extension selection. If you need different extensions, generate a new schematic via the Image Factory web UI.

#### Generating a Custom Schematic

If you need a different combination of extensions, you can generate a schematic programmatically:

```yaml
# schematic.yaml
customization:
  systemExtensions:
    officialExtensions:
      - siderolabs/iscsi-tools
      - siderolabs/qemu-guest-agent
      - siderolabs/tailscale
      - siderolabs/util-linux-tools
```

```bash
# Submit the schematic to get an ID
curl -X POST --data-binary @schematic.yaml \
  https://factory.talos.dev/schematics

# Response will contain the schematic ID to use in the image URL
```

This custom image ensures that the QEMU guest agent is available immediately after boot, allowing Proxmox to properly detect the VM's IP address and manage its lifecycle.

### Creating a Proxmox API Token

In Proxmox, create an API token for OpenTofu:

1. Go to **Datacenter → Permissions → API Tokens**
2. Click **Add** and create a token for your user
3. Note down the Token ID and Secret (the secret is only shown once)

## Project Structure

The OpenTofu configuration consists of the following files:

```
provision-vms/
├── main.tf              # Provider config and VM resources
├── variables.tf         # Variable declarations
├── outputs.tf           # Output definitions
├── terraform.tfvars     # Your actual configuration values
└── .gitignore           # Excludes sensitive files
```

## Configuration Files

### Provider Configuration (main.tf)

The main configuration file sets up the Proxmox provider and defines VM resources:

```hcl
terraform {
  required_providers {
    proxmox = {
      source  = "bpg/proxmox"
      version = "~> 0.85"
    }
  }
}

provider "proxmox" {
  endpoint  = var.proxmox_api_url
  api_token = "${var.proxmox_token_id}=${var.proxmox_token_secret}"
  insecure  = var.proxmox_tls_insecure

  ssh {
    agent = true
  }
}
```

#### Control Plane VM Resource

```hcl
resource "proxmox_virtual_environment_vm" "controlplane" {
  for_each = { for node in var.controlplane_nodes : node.name => node }

  name        = each.value.name
  description = "Talos Control Plane Node"
  tags        = lookup(each.value, "tags", ["talos", "controlplane", "kubernetes"])

  node_name = lookup(each.value, "target_node", var.default_proxmox_node)
  vm_id     = each.value.vmid

  machine       = "q35"
  scsi_hardware = "virtio-scsi-pci"
  bios          = var.bios_type

  stop_on_destroy     = true
  started             = true
  on_boot             = true
  tablet_device       = true
  keyboard_layout     = "en-us"
  reboot_after_update = false

  agent {
    enabled = true
  }

  cpu {
    type    = var.cpu_type
    sockets = lookup(each.value, "sockets", var.controlplane_defaults.sockets)
    cores   = lookup(each.value, "cores", var.controlplane_defaults.cores)
  }

  memory {
    dedicated = lookup(each.value, "memory", var.controlplane_defaults.memory)
    floating  = 0
  }

  cdrom {
    enabled   = true
    file_id   = "${var.iso_storage}:iso/${var.talos_iso}"
    interface = "ide2"
  }

  boot_order = ["ide2", "virtio0", "net0"]

  disk {
    datastore_id = lookup(each.value, "disk_storage", var.default_disk_storage)
    interface    = "virtio0"
    size         = lookup(each.value, "disk_size", var.controlplane_defaults.disk_size)
    file_format  = "raw"
    cache        = "writeback"
    aio          = "io_uring"
  }

  network_device {
    bridge      = lookup(each.value, "network_bridge", var.default_network_bridge)
    mac_address = each.value.mac_address
    model       = "virtio"
    vlan_id     = lookup(each.value, "vlan_tag", var.default_vlan_tag)
  }

  operating_system {
    type = "l26"
  }

  lifecycle {
    ignore_changes = [network_device]
  }
}
```

#### Worker VM Resource

Worker nodes support additional disks and network interfaces for storage-intensive workloads:

```hcl
resource "proxmox_virtual_environment_vm" "worker" {
  for_each = { for node in var.worker_nodes : node.name => node }

  name        = each.value.name
  description = "Talos Worker Node"
  tags        = lookup(each.value, "tags", ["talos", "worker", "kubernetes"])

  node_name = lookup(each.value, "target_node", var.default_proxmox_node)
  vm_id     = each.value.vmid

  # ... similar base configuration as control plane ...

  cpu {
    type    = var.cpu_type
    sockets = lookup(each.value, "sockets", var.worker_defaults.sockets)
    cores   = lookup(each.value, "cores", var.worker_defaults.cores)
  }

  memory {
    dedicated = lookup(each.value, "memory", var.worker_defaults.memory)
    floating  = 0
  }

  # Primary disk
  disk {
    datastore_id = lookup(each.value, "disk_storage", var.default_disk_storage)
    interface    = "virtio0"
    size         = lookup(each.value, "disk_size", var.worker_defaults.disk_size)
    file_format  = "raw"
    cache        = "writeback"
    aio          = "io_uring"
  }

  # Additional disks for storage (e.g., for Longhorn, Rook-Ceph)
  dynamic "disk" {
    for_each = lookup(each.value, "additional_disks", [])
    content {
      datastore_id = lookup(disk.value, "storage", var.default_disk_storage)
      interface    = "virtio${disk.key + 1}"
      size         = disk.value.size
      file_format  = "raw"
      cache        = "writeback"
      aio          = "io_uring"
    }
  }

  # Additional network interfaces
  dynamic "network_device" {
    for_each = lookup(each.value, "additional_networks", [])
    content {
      bridge  = network_device.value.bridge
      model   = "virtio"
      vlan_id = lookup(network_device.value, "vlan_tag", 0)
    }
  }
}
```

### Variable Definitions (variables.tf)

Define all configurable parameters:

```hcl
# Proxmox Connection
variable "proxmox_api_url" {
  description = "The URL of the Proxmox API"
  type        = string
}

variable "proxmox_token_id" {
  description = "The Proxmox API token ID"
  type        = string
}

variable "proxmox_token_secret" {
  description = "The Proxmox API token secret"
  type        = string
  sensitive   = true
}

variable "proxmox_tls_insecure" {
  description = "Skip TLS verification"
  type        = bool
  default     = true
}

# Default Settings
variable "default_proxmox_node" {
  description = "Default Proxmox node for VM placement"
  type        = string
  default     = "pve"
}

variable "default_disk_storage" {
  description = "Default storage for VM disks"
  type        = string
  default     = "local-lvm"
}

variable "default_network_bridge" {
  description = "Default network bridge"
  type        = string
  default     = "vmbr0"
}

variable "talos_iso" {
  description = "Talos ISO filename"
  type        = string
  default     = "talos-amd64.iso"
}

# Node Defaults
variable "controlplane_defaults" {
  description = "Default specs for control plane nodes"
  type = object({
    cores     = number
    sockets   = number
    memory    = number
    disk_size = number
  })
  default = {
    cores     = 4
    sockets   = 1
    memory    = 4096
    disk_size = 50
  }
}

variable "worker_defaults" {
  description = "Default specs for worker nodes"
  type = object({
    cores     = number
    sockets   = number
    memory    = number
    disk_size = number
  })
  default = {
    cores     = 4
    sockets   = 1
    memory    = 8192
    disk_size = 100
  }
}

# Node Lists
variable "controlplane_nodes" {
  description = "List of control plane node configurations"
  type        = list(any)
}

variable "worker_nodes" {
  description = "List of worker node configurations"
  type        = list(any)
}
```

### Outputs (outputs.tf)

Expose useful information about the created VMs:

```hcl
output "controlplane_nodes" {
  description = "Control plane node information"
  value = {
    for name, vm in proxmox_virtual_environment_vm.controlplane : name => {
      name        = vm.name
      vmid        = vm.vm_id
      target_node = vm.node_name
      ipv4        = vm.ipv4_addresses[1][0]
    }
  }
}

output "worker_nodes" {
  description = "Worker node information"
  value = {
    for name, vm in proxmox_virtual_environment_vm.worker : name => {
      name        = vm.name
      vmid        = vm.vm_id
      target_node = vm.node_name
      ipv4        = vm.ipv4_addresses[1][0]
    }
  }
}
```

## Example Configuration

Create a `terraform.tfvars` file with your specific configuration:

```hcl
# Proxmox Connection
proxmox_api_url      = "https://your-proxmox-server:8006"
proxmox_token_id     = "root@pam!opentofu"
proxmox_token_secret = "your-api-token-secret"

# Infrastructure Settings
default_proxmox_node   = "pve"
default_disk_storage   = "local-lvm"
default_network_bridge = "vmbr0"
iso_storage            = "local"
talos_iso              = "talos-v1.11.5-nocloud-amd64.iso"  # Custom image with extensions

# Control Plane Specs
controlplane_defaults = {
  cores     = 4
  sockets   = 1
  memory    = 4096
  disk_size = 50
}

# Worker Specs
worker_defaults = {
  cores     = 4
  sockets   = 1
  memory    = 8192
  disk_size = 100
}

# Control Plane Nodes
# MAC addresses are used for DHCP static lease assignments
controlplane_nodes = [
  {
    name        = "talos-cp-01"
    vmid        = 7001
    mac_address = "AA:BB:CC:11:22:01"
  },
  {
    name        = "talos-cp-02"
    vmid        = 7002
    mac_address = "AA:BB:CC:11:22:02"
  },
  {
    name        = "talos-cp-03"
    vmid        = 7003
    mac_address = "AA:BB:CC:11:22:03"
  }
]

# Worker Nodes with additional storage disks
worker_nodes = [
  {
    name        = "talos-worker-01"
    vmid        = 7011
    mac_address = "AA:BB:CC:11:22:11"
    cores       = 8
    memory      = 16384
    additional_disks = [
      { size = 100 }
    ]
  },
  {
    name        = "talos-worker-02"
    vmid        = 7012
    mac_address = "AA:BB:CC:11:22:12"
    cores       = 8
    memory      = 16384
    additional_disks = [
      { size = 100 }
    ]
  }
]
```

## DHCP Static Lease Configuration

Since we're using DHCP with static leases, you'll need to configure your DHCP server with reservations for each MAC address. Here's an example configuration for different DHCP servers:

### ISC DHCP Server

```conf
host talos-cp-01 {
  hardware ethernet AA:BB:CC:11:22:01;
  fixed-address 192.168.1.101;
}

host talos-cp-02 {
  hardware ethernet AA:BB:CC:11:22:02;
  fixed-address 192.168.1.102;
}

host talos-cp-03 {
  hardware ethernet AA:BB:CC:11:22:03;
  fixed-address 192.168.1.103;
}

host talos-worker-01 {
  hardware ethernet AA:BB:CC:11:22:11;
  fixed-address 192.168.1.111;
}

host talos-worker-02 {
  hardware ethernet AA:BB:CC:11:22:12;
  fixed-address 192.168.1.112;
}
```

### OPNsense/pfSense

Navigate to **Services → DHCPv4 → [Your Interface] → Static Mappings** and add an entry for each MAC address with the desired IP.

### Unifi Dream Machine

Go to **Settings → Networks → [Your Network] → DHCP → Static IP** and create reservations for each node.

## Deployment Steps

### 1. Initialize OpenTofu

```bash
# Initialize the working directory
tofu init
```

### 2. Review the Execution Plan

```bash
# Preview what will be created
tofu plan
```

### 3. Apply the Configuration

```bash
# Create the infrastructure
tofu apply
```

### 4. Verify VM Creation

After successful deployment, OpenTofu will output the node information:

```
Outputs:

controlplane_nodes = {
  "talos-cp-01" = {
    ipv4        = "192.168.1.101"
    name        = "talos-cp-01"
    target_node = "pve"
    vmid        = 7001
  }
  # ... additional nodes
}
```

## Bootstrapping the Talos Cluster

After the VMs are provisioned, bootstrap the Talos cluster:

### 1. Generate Talos Configuration

When generating the configuration, use the `--install-image` flag to specify the factory installer image with your extensions. This ensures that when nodes are installed, they use the same extensions as the boot ISO:

```bash
# Generate cluster secrets first (keep this safe!)
talosctl gen secrets -o secrets.yaml

# Generate cluster configuration with factory installer image
talosctl gen config my-cluster https://192.168.1.100:6443 \
  --with-secrets secrets.yaml \
  --output-dir _out \
  --install-image factory.talos.dev/installer/077514df2c1b6436460bc60faabc976687b16193b8a1290fda4366c69024fec2:v1.11.5
```

> **Important:** The `--install-image` uses the same schematic ID as your ISO, but with the `/installer/` path instead of `/image/`. This ensures the installed system has all the same extensions.

### 2. Apply Configuration to Nodes

```bash
# Apply to control plane nodes
talosctl apply-config --insecure \
  --nodes 192.168.1.101 \
  --file _out/controlplane.yaml

talosctl apply-config --insecure \
  --nodes 192.168.1.102 \
  --file _out/controlplane.yaml

talosctl apply-config --insecure \
  --nodes 192.168.1.103 \
  --file _out/controlplane.yaml

# Apply to worker nodes
talosctl apply-config --insecure \
  --nodes 192.168.1.111 \
  --file _out/worker.yaml

talosctl apply-config --insecure \
  --nodes 192.168.1.112 \
  --file _out/worker.yaml
```

### 3. Bootstrap the Cluster

```bash
# Configure talosctl endpoint
export TALOSCONFIG="_out/talosconfig"
talosctl config endpoint 192.168.1.101
talosctl config node 192.168.1.101

# Bootstrap etcd on the first control plane node
talosctl bootstrap
```

### 4. Retrieve Kubeconfig

```bash
# Get the kubeconfig file
talosctl kubeconfig ./kubeconfig

# Verify cluster access
export KUBECONFIG=./kubeconfig
kubectl get nodes
```

## Managing the Infrastructure

### Scaling the Cluster

To add more nodes, simply update `terraform.tfvars`:

```hcl
worker_nodes = [
  # ... existing nodes ...
  {
    name        = "talos-worker-03"
    vmid        = 7013
    mac_address = "AA:BB:CC:11:22:13"
    cores       = 8
    memory      = 16384
  }
]
```

Then apply the changes:

```bash
tofu apply
```

### Destroying the Infrastructure

```bash
# Remove all VMs
tofu destroy
```

## Best Practices

1. **Version Control**: Keep your `.tfvars` files out of version control (they may contain sensitive data). Use `.tfvars.example` as a template.

2. **State Management**: For production, use remote state storage (S3, Consul, etc.) instead of local state files.

3. **Separate Environments**: Create separate `.tfvars` files for different environments (dev, staging, production).

4. **Resource Tagging**: Use consistent tags for resource organization and cost tracking.

5. **Network Security**: Place your cluster in a dedicated VLAN with appropriate firewall rules.

## Conclusion

Using OpenTofu to provision Talos Kubernetes clusters on Proxmox provides a repeatable, version-controlled approach to infrastructure management. The combination of Talos's security-focused design and infrastructure as code practices creates a solid foundation for running production Kubernetes workloads.

The DHCP static lease approach simplifies network management while maintaining consistent addressing—making it easy to automate cluster bootstrapping and DNS configuration.

## Resources

- [Talos Linux Documentation](https://www.talos.dev/docs/)
- [OpenTofu Documentation](https://opentofu.org/docs/)
- [BPG Proxmox Provider](https://registry.terraform.io/providers/bpg/proxmox/latest/docs)
- [Proxmox VE Documentation](https://pve.proxmox.com/pve-docs/)
