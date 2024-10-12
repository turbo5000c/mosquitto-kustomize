# Mosquitto Kustomize Deployment

This repository contains a Kustomize setup for deploying [Eclipse Mosquitto](https://mosquitto.org/), a lightweight MQTT broker, on Kubernetes. This guide provides instructions for deploying Mosquitto using Kustomize with FluxCD.

## Prerequisites

- A Kubernetes cluster (tested with k0s, k3s, and microk8s)
- Kustomize installed ([installation guide](https://kubectl.docs.kubernetes.io/installation/kustomize/))
- FluxCD installed and configured for GitOps ([installation guide](https://fluxcd.io/docs/installation/)) (optional)

## Overview

The setup includes:
- A Kubernetes Deployment for Mosquitto
- Service definitions for accessing Mosquitto
- Persistent Volume Claims (PVC) for data storage
- ConfigMap for Mosquitto configuration
- Renovate configuration for dependency management

## Configuration

### 1. Clone the Repository

Clone this repository to your local machine:
```
    git clone https://github.com/turbo5000c/mosquitto-kustomize.git
    cd mosquitto-kustomize
```
### 2. Update Configuration Files

The following configuration files may need adjustments based on your environment:
- `kustomization.yaml`: Manages resources with Kustomize.
- `configmap.yaml`: Contains configuration settings for Mosquitto, such as `mosquitto.conf` parameters.
- `deployment.yaml`: Defines the Mosquitto deployment, including image version and replica settings.
- `pvc.yaml`: Specifies Persistent Volume Claim for data storage, including access modes and storage requirements.
- `service.yaml`: Defines the Service to expose Mosquitto, defaulted to ClusterIP.
- `renovate.json`: Manages dependencies with Renovate. Adjust settings if required for dependency updates.

### 3. Deploying with Kustomize

To apply the configuration with Kustomize, run:
```
    kubectl apply -k .
```
### 4. Deploying with FluxCD

If using FluxCD for GitOps:
1. Ensure your FluxCD setup is configured to watch this repository.
2. Add the repository to your Flux configuration by referencing it in `kustomization.yaml`.
3. Flux will automatically detect and apply changes to your cluster.

## Namespace Consideration

Resources are deployed by default into the `mosquitto` namespace. Change this by editing `metadata.namespace` in `kustomization.yaml`, and create the namespace if it doesn’t exist:
```
    kubectl create namespace <your-namespace>
```
## Storage Configuration

If persistent data storage is needed, the PVC configuration in `pvc.yaml` can be modified. Update the `storageClassName` and `storage` values as required. Example configuration:
```
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mosquitto-pvc
      namespace: mosquitto
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
```
Adjust `storageClassName` and `storage` values based on your requirements.

## Accessing Mosquitto

By default, the `service.yaml` file exposes Mosquitto using the `ClusterIP` service type. Modify it to `NodePort` or `LoadBalancer` as needed for external access.

To access Mosquitto via NodePort, retrieve the assigned port with:
```
    kubectl get svc -n mosquitto
```
Use an MQTT client to connect via `mqtt://<NODE_IP>:<PORT>`.

## Uninstall

To remove the Mosquitto deployment and all related resources, run:
```
    kubectl delete -k .
```
Or, if using FluxCD, remove the repository or folder from your Flux configuration, and Flux will delete the resources.

## Additional Notes

- **Logging**: View logs with:
```
      kubectl logs -f <pod-name> -n mosquitto
```
  Replace `<pod-name>` with the Mosquitto pod’s name.

- **Configuration**: Update `configmap.yaml` for additional settings such as ports, authentication, and other MQTT options.

## License

This repository is open-source and available under the MIT License.
