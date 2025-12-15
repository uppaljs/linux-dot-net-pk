---
title: "Setting Up HAProxy Load Balancer for Kubernetes and Talos API"
slug: haproxy-load-balancer-talos-kubernetes
date: 2025-12-15
categories:
  - Howtos
  - Kubernetes
tags:
  - haproxy
  - kubernetes
  - talos
  - load-balancer
  - high-availability
  - homelab
draft: false
---

When running a highly available Kubernetes cluster with multiple control plane nodes, you need a load balancer to distribute API traffic across all control plane endpoints. This guide walks through setting up HAProxy 3.2 on Debian to load balance both the Kubernetes API (port 6443) and Talos API (port 50000).

## Architecture Overview

| Component | Address | Port | Purpose |
|-----------|---------|------|---------|
| HAProxy LB | 192.168.66.160 | 6443 | Kubernetes API |
| HAProxy LB | 192.168.66.160 | 50000 | Talos API |
| HAProxy LB | 192.168.66.160 | 9600 | Stats Dashboard |
| Control Plane 1 | 192.168.66.161 | 6443/50000 | talos-lon-cp01 |
| Control Plane 2 | 192.168.66.162 | 6443/50000 | talos-lon-cp02 |
| Control Plane 3 | 192.168.66.163 | 6443/50000 | talos-lon-cp03 |

## Prerequisites

- Debian Trixie (or compatible) server for the load balancer
- Network connectivity to all control plane nodes
- Root or sudo access on the load balancer server

## Install HAProxy 3.2

HAProxy 3.2 is available from the official HAProxy Debian repository. First, add the repository signing key and apt source:

```bash
curl https://haproxy.debian.net/haproxy-archive-keyring.gpg \
    --create-dirs --output /etc/apt/keyrings/haproxy-archive-keyring.gpg

echo 'deb "[signed-by=/etc/apt/keyrings/haproxy-archive-keyring.gpg]" \
    https://haproxy.debian.net trixie-backports-3.2 main' \
    > /etc/apt/sources.list.d/haproxy.list
```

Update the package cache and install HAProxy:

```bash
apt update -y
apt install haproxy=3.2.\*
```

Enable and start the HAProxy service:

```bash
systemctl enable haproxy
systemctl start haproxy
```

Verify HAProxy is running:

```bash
systemctl status haproxy
```

## Configure HAProxy

Edit the HAProxy configuration file at `/etc/haproxy/haproxy.cfg`. Replace the default configuration with the following:

```conf
global
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon
    # Default SSL material locations
    ca-base /etc/ssl/certs
    crt-base /etc/ssl/private
    # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
    ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
    ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
    ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http

#---------------------------------------------------------------------
# HAProxy Stats Page
#---------------------------------------------------------------------
listen stats
    bind 192.168.66.160:9600
    mode http
    stats enable
    stats uri /
    stats refresh 10s
    stats auth admin:your-secure-password
    acl allowed_networks src 172.16.25.0/24
    http-request deny if !allowed_networks

#---------------------------------------------------------------------
# apiserver frontend which proxys to the control plane nodes
#---------------------------------------------------------------------
frontend k8s_apiserver
    bind 192.168.66.160:6443
    mode tcp
    option tcplog
    default_backend k8s_controlplane

frontend talos_apiserver
    bind 192.168.66.160:50000
    mode tcp
    option tcplog
    default_backend talos_controlplane

#---------------------------------------------------------------------
# round robin balancing for apiserver
#---------------------------------------------------------------------
backend k8s_controlplane
    option httpchk GET /healthz
    http-check expect status 200
    mode tcp
    option ssl-hello-chk
    balance     roundrobin
        server talos-lon-cp01 192.168.66.161:6443 check
        server talos-lon-cp02 192.168.66.162:6443 check
        server talos-lon-cp03 192.168.66.163:6443 check

backend talos_controlplane
    option httpchk GET /healthz
    http-check expect status 200
    mode tcp
    option ssl-hello-chk
    balance     roundrobin
        server talos-lon-cp01 192.168.66.161:50000 check
        server talos-lon-cp02 192.168.66.162:50000 check
        server talos-lon-cp03 192.168.66.163:50000 check
```

## Configuration Breakdown

### Global Section

The `global` section configures HAProxy process-wide settings:
- **chroot**: Runs HAProxy in a chroot jail for security
- **stats socket**: Enables runtime API for dynamic configuration
- **ssl-default-bind-***: Sets secure TLS defaults following Mozilla's intermediate profile

### Defaults Section

The `defaults` section sets values inherited by all frontends and backends:
- **mode http**: Default mode (overridden to TCP for our API proxying)
- **timeout connect/client/server**: Connection timeout values in milliseconds

### Stats Dashboard

The `stats` listener provides a web-based dashboard to monitor HAProxy:
- Bound to port 9600 on the load balancer IP
- Protected with basic authentication
- Restricted to specific networks via ACL
- Auto-refreshes every 10 seconds

> **Security Note**: Update the `stats auth` password and `allowed_networks` ACL to match your environment. Consider using a more restrictive network range.

### Kubernetes API Frontend/Backend

The `k8s_apiserver` frontend and `k8s_controlplane` backend handle Kubernetes API traffic:
- **mode tcp**: TCP passthrough (required for TLS termination at the API server)
- **option ssl-hello-chk**: Health checks using SSL hello handshake
- **option httpchk GET /healthz**: HTTP health check endpoint
- **balance roundrobin**: Distributes requests evenly across servers

### Talos API Frontend/Backend

The `talos_apiserver` frontend and `talos_controlplane` backend follow the same pattern for Talos API traffic on port 50000.

## Apply Configuration

Validate the configuration syntax:

```bash
haproxy -c -f /etc/haproxy/haproxy.cfg
```

Restart HAProxy to apply changes:

```bash
systemctl restart haproxy
```

## Verify the Setup

Check that HAProxy is listening on the configured ports:

```bash
netstat -tnulp | grep haproxy
```

Expected output:

```
tcp    0    0 192.168.66.160:9600     0.0.0.0:*    LISTEN    5712/haproxy
tcp    0    0 192.168.66.160:6443     0.0.0.0:*    LISTEN    5712/haproxy
tcp    0    0 192.168.66.160:50000    0.0.0.0:*    LISTEN    5712/haproxy
```

Test connectivity to the Kubernetes API through the load balancer:

```bash
curl -k https://192.168.66.160:6443/healthz
```

## Configure Talos and kubectl

Update your `talosconfig` to use the load balancer endpoint:

```bash
talosctl config endpoint 192.168.66.160
```

When generating Talos cluster configuration, use the load balancer IP as the endpoint:

```bash
talosctl gen config my-cluster https://192.168.66.160:6443
```

Your kubeconfig should point to the load balancer:

```yaml
clusters:
- cluster:
    server: https://192.168.66.160:6443
  name: my-cluster
```

## Monitoring

Access the HAProxy stats dashboard at:

```
http://192.168.66.160:9600/
```

The dashboard shows:
- Backend server health status (green/red)
- Current connections and request rates
- Bytes in/out per server
- Response time metrics

## High Availability Considerations

For production environments, consider these enhancements:

1. **Redundant Load Balancers**: Deploy multiple HAProxy instances with keepalived or a cloud load balancer in front
2. **Health Check Tuning**: Adjust `inter`, `fall`, and `rise` parameters for faster failover:
   ```conf
   server talos-lon-cp01 192.168.66.161:6443 check inter 2000 fall 3 rise 2
   ```
3. **Connection Limits**: Set `maxconn` to prevent overload
4. **Logging**: Configure rsyslog to capture HAProxy logs for troubleshooting

## Summary

You now have HAProxy 3.2 configured to load balance:

- **Kubernetes API** on port 6443 across 3 control plane nodes
- **Talos API** on port 50000 across 3 control plane nodes
- **Stats dashboard** on port 9600 for monitoring

This setup provides a single stable endpoint for your Talos Kubernetes cluster, enabling high availability and simplified client configuration.
