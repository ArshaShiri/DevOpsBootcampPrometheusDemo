## DevOps Bootcamp Prometheus Demo

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

## Prometheus Web UI

    # After executing `kubectl get all -n monitoring`, we can check the service 
    # that exposes Prometheus UI: service/monitoring-kube-prometheus-prometheus
    # To be able to access it locally:
    kubectl port-forward service/monitoring-kube-prometheus-prometheus -n monitoring 9090:9090 &

    # We can then use the given address to access the UI from our browser
    
## Grafana
    # We port forward service/monitoring-grafana 
    kubectl port-forward service/monitoring-grafana -n monitoring 8080:80 &

The default username and password are `admin` and `prom-operator` respectively.
