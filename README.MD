
# Flux Configuration for Kubernetes

This repository contains Flux configurations for managing Helm releases of various applications on a Kubernetes cluster. The configurations include **Pi-hole** and **Home Assistant** deployments using Flux's `HelmRelease` resources.

## Prerequisites

Before using this repository, ensure the following are in place:

- Kubernetes cluster with Flux v2 installed
- Flux controllers: `HelmController`, `KustomizeController`, and `SourceController`
- Helm repositories for Pi-hole (`mojo2600`) and Home Assistant (`home-assistant-repo`)

## Contents

### 1. `pihole-helmrelease.yaml`

This file contains the HelmRelease configuration for deploying **Pi-hole**, an ad-blocking and DNS management application, on your Kubernetes cluster.

#### Configuration Options

##### Create password
kubectl create secret generic pihole-secret \
    --from-literal='password=<ADMIN_PASSWORD>' \
    --namespace pihole

- **DNS Servers**: Configured to use Google's DNS servers (`8.8.8.8` and `8.8.4.4`).
- **Persistent Volume**: Configures persistent storage with `5Gi` for Pi-hole's data.
- **Admin Password**: Dont be fooled, im using a k8s secret called pihole-secret (created manually)
- **Ingress**: Ingress configuration is disabled by default.
- **Monitoring**: Option to enable a Pi-hole exporter for monitoring with Prometheus (disabled by default).
- **Service Type**: The Pi-hole web UI is exposed via a LoadBalancer service on ports `80` (HTTP) and `443` (HTTPS).

#### Example Values:
```yaml
DNS1: 8.8.8.8
DNS2: 8.8.4.4
admin:
  existingSecret: pihole-secret
  passwordKey: password
persistentVolumeClaim:
  size: 5Gi
serviceWeb:
  type: LoadBalancer
  port: 80
  https:
    enabled: true
    port: 443
```

### 2. `home-assistant-helmrelease.yaml`

This file contains the HelmRelease configuration for deploying **Home Assistant**, a popular open-source home automation platform, on Kubernetes.

#### Configuration Options

- **Persistence**: Configured with a persistent volume of `5Gi` using the `nfs-client` storage class.
- **Add-ons**: Includes a configuration for enabling `codeserver` (a web-based code editor).
- **Networking**: Configured to use `hostNetwork: true`, which allows the Home Assistant container to use the host network.
- **Service**: Exposes Home Assistant as a LoadBalancer service on port `8123`.

#### Example Values:
```yaml
addons:
  codeserver:
    enabled: enable
    service:
      type: LoadBalancer
persistence:
  size: 5Gi
  storageClass: nfs-client
hostNetwork: true
```

## Flux Setup

1. **Install Flux**: Ensure that Flux v2 is installed and running in your Kubernetes cluster. For more information, follow the [Flux installation guide](https://fluxcd.io/docs/installation/).
2. **Add Helm Repositories**: Before applying the Helm releases, make sure to add the required Helm repositories to Flux:

   ```bash
   flux create source helm mojo2600      --url=https://mojo2600.github.io/helm-charts      --interval=1h      --export > helm-repository-mojo2600.yaml

   flux create source helm home-assistant-repo      --url=https://github.com/home-assistant/helm-charts      --interval=1h      --export > helm-repository-home-assistant.yaml
   ```

3. **Apply Flux Configurations**:
   Apply the Flux configuration to your cluster using the following command:

   ```bash
   kubectl apply -f pihole-helmrelease.yaml
   kubectl apply -f home-assistant-helmrelease.yaml
   ```

   Flux will automatically synchronize these resources with the cluster, ensuring the correct Helm releases are installed and managed.

## Resources

- [Pi-hole Helm Chart](https://github.com/mojo2600/helm-charts/tree/master/charts/pihole)
- [Home Assistant Helm Chart](https://github.com/home-assistant/helm-charts/tree/main/charts/home-assistant)
