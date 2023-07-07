# DevOps Bootcamp Prometheus Demo

## Setting up the EKS

    # We create a default cluster in the default region
    eksctl create cluster

    kubectl get node

    # We can deploy the microservices
    kubectl apply -f config.yaml

## Deploying Prometheus Stack Using Helm

We follow the instructions presented [here](https://github.com/prometheus-community/helm-charts).

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update

    # Create a separate namespace for the monitoring part
    kubectl create namespace monitoring

    helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring

    # After installation we can check the status of the pod
    kubectl --namespace monitoring get pods -l "release=monitoring"

    # Alternatively, we can check everything under monitoring namespace
    kubectl get all -n monitoring
