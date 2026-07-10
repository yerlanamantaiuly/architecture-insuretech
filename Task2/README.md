# Задание 2. Динамическое масштабирование контейнеров

## Часть 1. Масштабирование по памяти

Запуск локального кластера и metrics-server:

```bash
minikube start
minikube addons enable metrics-server
```

Применение приложения, сервиса и HPA:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa-memory.yaml
kubectl get hpa insuretech-demo-app-memory --watch
```

Генерация нагрузки:

```bash
kubectl port-forward service/insuretech-demo-app 8080:8080
locust -f locustfile.py --host http://localhost:8080
```

Команды для сохранения доказательств масштабирования:

```bash
kubectl describe hpa insuretech-demo-app-memory > evidence-memory-hpa.txt
kubectl get deployment insuretech-demo-app -o wide >> evidence-memory-hpa.txt
kubectl get pods -l app=insuretech-demo-app -o wide >> evidence-memory-hpa.txt
```

## Часть 2. Масштабирование по количеству запросов в секунду

Установка Prometheus и Prometheus Adapter:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus -f prometheus-values.yaml
helm install prometheus-adapter prometheus-community/prometheus-adapter -f prometheus-adapter-values.yaml
kubectl apply -f hpa-rps.yaml
```

Проверка сбора метрик:

```bash
kubectl port-forward service/prometheus-server 9090:80
```

В Prometheus Web UI выполнить запрос:

```promql
sum(rate(http_requests_total[1m])) by (pod)
```

Проверка пользовательской метрики и HPA:

```bash
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/http_requests_per_second"
kubectl get hpa insuretech-demo-app-rps --watch
```

Команды для сохранения доказательств масштабирования:

```bash
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/http_requests_per_second" > evidence-rps-hpa.txt
kubectl describe hpa insuretech-demo-app-rps >> evidence-rps-hpa.txt
kubectl get deployment insuretech-demo-app -o wide >> evidence-rps-hpa.txt
kubectl get pods -l app=insuretech-demo-app -o wide >> evidence-rps-hpa.txt
```

В качестве доказательств для сдачи можно приложить либо скриншоты Kubernetes Dashboard и Prometheus UI, либо текстовые файлы `evidence-memory-hpa.txt` и `evidence-rps-hpa.txt`, полученные командами выше.
