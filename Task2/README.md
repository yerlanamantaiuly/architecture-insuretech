# Task2: Dynamic autoscaling

## Memory-based autoscaling

Apply the demo application, service, and memory HPA:

```bash
minikube start
minikube addons enable metrics-server
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa-memory.yaml
kubectl get hpa insuretech-demo-app-memory --watch
```

Generate load:

```bash
kubectl port-forward service/insuretech-demo-app 8080:8080
locust -f locustfile.py --host http://localhost:8080
```

Expected evidence to save after the test:

```bash
kubectl describe hpa insuretech-demo-app-memory
kubectl get deployment insuretech-demo-app
kubectl get pods -l app=insuretech-demo-app
```

## RPS-based autoscaling

Install Prometheus and Prometheus Adapter, then configure the custom metric:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus
helm install prometheus-adapter prometheus-community/prometheus-adapter -f prometheus-adapter-values.yaml
kubectl apply -f prometheus-scrape-config.yaml
kubectl apply -f hpa-rps.yaml
```

Check that application metrics are scraped:

```bash
kubectl port-forward service/prometheus-server 9090:80
```

Open Prometheus Web UI and run:

```promql
sum(rate(http_requests_total[1m])) by (pod)
```

Check the custom metric exposed to HPA:

```bash
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/http_requests_per_second"
kubectl get hpa insuretech-demo-app-rps --watch
```

Save the Prometheus screenshot and HPA/deployment output after generating load with Locust.
