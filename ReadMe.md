# Ingress and Horizontal Pod Autoscaler (HPA) with AKS Cluster Installation Guide

This guide provides a comprehensive overview of setting up an Azure Kubernetes Service (AKS) cluster, configuring Ingress, and implementing Horizontal Pod Autoscaler (HPA). The instructions include command-line steps and relevant images to facilitate understanding.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Creating an AKS Cluster](#creating-an-aks-cluster)
3. [Configuring Ingress](#configuring-ingress)
4. [Setting Up Horizontal Pod Autoscaler (HPA)](#setting-up-horizontal-pod-autoscaler-hpa)

## Prerequisites

Before proceeding, ensure you have the following:
- An **Azure account**.
- The **Azure CLI** installed.
- **kubectl** installed for managing Kubernetes clusters.
- Familiarity with Kubernetes concepts.

## Creating an AKS Cluster

To create an AKS cluster, follow these steps:

1. **Log in to Azure**:
   ```bash
   az login
   ```

2. **Set the subscription** (if necessary):
   ```bash
   az account set --subscription "Your Subscription Name"
   ```

3. **Create a resource group**:
   ```bash
   az group create --name myResourceGroup --location eastus
   ```

4. **Create the AKS cluster**:
   ```bash
   az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 1 --enable-addons monitoring --generate-ssh-keys
   ```

5. **Connect to the cluster**:
   ```bash
   az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
   ```

6. **Verify the connection**:
   ```bash
   kubectl get nodes
   ```

AKS Cluster Creation

## Configuring Ingress

Ingress allows you to manage external access to your services in a Kubernetes cluster.

1. **Install NGINX Ingress Controller**:
   ```bash
   helm repo add ingress-nginx https://charts.ingress-nginx.io
   helm repo update
   helm install nginx-ingress ingress-nginx/ingress-nginx
   ```

2. **Verify the installation**:
   ```bash
   kubectl get pods -n ingress-nginx
   ```

3. **Create an Ingress resource**:
   Create a file named `my-ingress.yaml`:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: my-ingress
     annotations:
       nginx.ingress.kubernetes.io/rewrite-target: /
   spec:
     rules:
       - host: myapp.example.com
         http:
           paths:
             - path: /
               pathType: Prefix
               backend:
                 service:
                   name: my-service
                   port:
                     number: 80
   ```

4. **Apply the Ingress configuration**:
   ```bash
   kubectl apply -f my-ingress.yaml
   ```

NGINX Ingress Controller

## Setting Up Horizontal Pod Autoscaler (HPA)

HPA automatically scales the number of pods in a deployment based on CPU utilization or other select metrics.

1. **Ensure Metrics Server is installed**:
   ```bash
   kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   ```

2. **Create a deployment for HPA**:
   Create a file named `my-deployment.yaml`:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-deployment
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: my-app
     template:
       metadata:
         labels:
           app: my-app
       spec:
         containers:
           - name: my-container
             image: my-image:v1
             resources:
               requests:
                 cpu: 200m
                 memory: 512Mi
               limits:
                 cpu: 500m
                 memory: 1Gi
   ```

3. **Apply the deployment**:
   ```bash
   kubectl apply -f my-deployment.yaml
   ```

4. **Create HPA resource**:
   Create a file named `my-hpa.yaml`:
   ```yaml
   apiVersion: autoscaling/v2beta2
   kind: HorizontalPodAutoscaler
   metadata:
     name: my-hpa
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: my-deployment
     minReplicas: 1
     maxReplicas: 10
     metrics:
       - type: Resource
         resource:
           name: cpu
           target:
             type: Utilization 
             averageUtilization: 50 
   ```

5. **Apply the HPA configuration**:
   ```bash
   kubectl apply -f my-hpa.yaml 
   ```

