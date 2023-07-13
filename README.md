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


![image](https://github.com/ArshaShiri/DevOpsBootcampPrometheusDemo/assets/18715119/898b02d3-ef4e-4bc2-9fc3-722d84784ca4)


In order to simulate a spike in CPU usage we deploy a simple pod that can create a lot of requests which results in cpu spike:

    # This will put us inside the container where we can make 
    # dummy request to our online shop service to spike the CPU usage
    kubectl run curl-test --image=radial/busyboxplus:curl -i --tty --rm

    # We check the end point of the online shop service
    kubectl get svc
    
        # frontend-external       LoadBalancer   10.100.150.77    aad02b72ec2c747ec9f63f87d324a84e-273112622.eu-central-1.elb.amazonaws.com

    # In the container that we just ran we create a simple script:
    vi test.sh

    for i in $(seq 1 100000)
    do
      curl http://aad02b72ec2c747ec9f63f87d324a84e-273112622.eu-central-1.elb.amazonaws.com > test.txt
    done

    chmod +x test.sh
    ./test.sh

After running the script, the CPU spike can be observer in Grafana:

![image](https://github.com/ArshaShiri/DevOpsBootcampPrometheusDemo/assets/18715119/89d4282b-dd64-4528-b0a2-8cbedb70f726)

Also, the frontend service can be observed taking a lot of resources:

![image](https://github.com/ArshaShiri/DevOpsBootcampPrometheusDemo/assets/18715119/73a85ac0-f508-442f-a3bf-dcb885bd1ad1)


## Alert Rules in Prometheus

![image](https://github.com/ArshaShiri/DevOpsBootcampPrometheusDemo/assets/18715119/68f16f97-7df4-41e3-9128-2b5985ee1719)

We now create two new alert rules. One fires when the CPU usage exceeds 50% and the other when a pod cannot start.

After creating `alert-rules.yaml` we can apply it by `kubectl apply -f alert-rules.yaml`.

We can now get the newly created rule:

    # We can see main-rules in the list
    kubectl get PrometheusRule -n monitoring

    kubectl get pod -n monitoring

    kubectl logs prometheus-monitoring-kube-prometheus-prometheus-0 -n monitoring -c config-reloader