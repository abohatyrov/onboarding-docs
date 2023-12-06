# Comprehensive Guide: Setting up Prometheus Monitoring with Minikube

## Introduction

This comprehensive guide provides step-by-step instructions for setting up Prometheus monitoring with Minikube. Prometheus is a robust open-source monitoring and alerting toolkit designed for containerized environments. The guide aims to enable users, including non-technical individuals, to monitor the status of a Docker container and a virtual machine.

### Understanding the Components

Before diving into the setup process, let's briefly understand the key components involved:

#### Minikube

Minikube is a tool that simplifies the local deployment of Kubernetes clusters. It allows users to run a single-node Kubernetes cluster on their local machines for development and testing purposes.

#### Kubernetes

Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications.

#### Prometheus

Prometheus is a comprehensive monitoring and alerting system. It collects time-series data, allowing users to query and visualize metrics to ensure the health and performance of applications.

### Prerequisites

Ensure the following software is installed on your machine:

- [Vagrant](https://www.vagrantup.com/)
- [VirtualBox](https://www.virtualbox.org/)
- [Minikube](https://minikube.sigs.k8s.io/)

## Configuration

### Vagrantfile Configuration

The Vagrantfile serves as the configuration file for Vagrant, facilitating the provisioning of a virtual machine with specific settings. Below is an annotated version of the Vagrantfile:

```ruby
# Existing configurations...

# Add configurations for Prometheus
config.vm.provision "file", source: "/path/to/prometheusClusterRole.yml", destination: "/home/vagrant/prometheus/prometheusClusterRole.yml"
config.vm.provision "file", source: "/path/to/prometheusConfigMap.yml", destination: "/home/vagrant/prometheus/prometheusConfigMap.yml"
config.vm.provision "file", source: "/path/to/prometheusDeployment.yml", destination: "/home/vagrant/prometheus/prometheusDeployment.yml"

# Existing configurations...

# Shell provisioning script
config.vm.provision "shell", inline: <<-SHELL
  # Existing commands...

  # Install Docker, Kubernetes tools, and Minikube
  # ...

  # Create Kubernetes namespace for monitoring
  sudo kubectl create namespace monitoring

  # Apply Prometheus configurations
  sudo kubectl apply -f /home/vagrant/prometheus

  # Wait for Prometheus to start
  sleep 120

  # Port forward to access Prometheus dashboard
  sudo kubectl port-forward $(sudo kubectl get pods -n monitoring --output=jsonpath={.items..metadata.name} --selector=app=prometheus-server) --address 192.168.0.154 9090:9090 -n monitoring
SHELL
end
```

### Shell Provisioning Script

This script orchestrates the setup within the virtual machine. It installs Docker, Kubernetes tools, and Minikube. It then creates a Kubernetes namespace for monitoring, applies Prometheus configurations, and sets up port forwarding for accessing the Prometheus dashboard.

## Detailed Prometheus Configurations

### `prometheusClusterRole.yml`

This YAML file defines a Kubernetes `ClusterRole` and `ClusterRoleBinding` necessary for Prometheus to access and collect metrics from various Kubernetes resources.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - nodes/proxy
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: default
  namespace: monitoring
```

#### Explanation

The `ClusterRole` and `ClusterRoleBinding` combination grants Prometheus the necessary permissions to access and collect metrics from nodes, services, endpoints, pods, ingresses, and the `/metrics` path within the `monitoring` namespace.

### `prometheusConfigMap.yml`

This YAML file defines a Kubernetes `ConfigMap` resource encapsulating the Prometheus configuration file (`prometheus.yml`) and alerting rules.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-server-conf
  labels:
    name: prometheus-server-conf
  namespace: monitoring
data:
  prometheus.yml: |-
    # Global configuration for Prometheus
    global:
      scrape_interval: 10s

    # Define scrape configurations for various jobs
    scrape_configs:
      - job_name: 'prometheus_master'
        scrape_interval: 5s
        static_configs:
          - targets: ['localhost:9090']
      - job_name: 'wordpress_node'
        static_configs:
          - targets: ['54.146.199.206:9100']
      # ... (Additional job configurations)
```

#### Explanation

The `ConfigMap` includes the Prometheus configuration file (`prometheus.yml`). The configuration specifies global settings like scrape intervals and defines scrape configurations for various jobs, allowing Prometheus to collect metrics from specific targets.

### `prometheusDeployment.yml`

This YAML file defines a Kubernetes `Deployment` resource for deploying the Prometheus server as a container.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus-deployment
  namespace: monitoring
  labels:
    app: prometheus-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus-server
  template:
    metadata:
      labels:
        app: prometheus-server
   

 spec:
      containers:
        - name: prometheus
          image: prom/prometheus
          args:
            - "--storage.tsdb.retention.time=12h"
            - "--config.file=/etc/prometheus/prometheus.yml"
            - "--storage.tsdb.path=/prometheus/"
          ports:
            - containerPort: 9090
          resources:
            requests:
              cpu: 500m
              memory: 500M
            limits:
              cpu: 1
              memory: 1Gi
          volumeMounts:
            - name: prometheus-config-volume
              mountPath: /etc/prometheus/
            - name: prometheus-storage-volume
              mountPath: /prometheus/
      volumes:
        - name: prometheus-config-volume
          configMap:
            defaultMode: 420
            name: prometheus-server-conf

        - name: prometheus-storage-volume
          emptyDir: {}
```

#### Explanation

The `Deployment` resource ensures that there is one instance of the Prometheus server running. It specifies the container image, resource requirements, and volume mounts for configuration and storage.

## Conclusion

This guide has empowered you to configure Minikube for Prometheus monitoring of a Docker container and a virtual machine. The detailed explanations and comprehensive configurations should enhance understanding, even for individuals with limited technical background. The `prometheusClusterRole.yml`, `prometheusConfigMap.yml`, and `prometheusDeployment.yml` files collectively define the necessary Kubernetes resources for Prometheus to function within the specified environment.