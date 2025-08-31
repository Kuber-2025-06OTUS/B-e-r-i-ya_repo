# Установка Prometheus

## Добавляем репозиторий prometheus-community

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
## Создаем namespace для мониторинга
```kubectl create namespace monitoring```

## Устанавливаем kube-prometheus-stack
```
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \
  --set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false
```

# Запускаем кастомный nginx и nginx-exporter 

```
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-exporter-deployment.yaml
kubectl apply -f servicemonitor.yaml
```

# Проверка


## Проверяем поды
```
kubectl get pods -l app=nginx
kubectl get pods -l app=nginx-exporter
```

## Проверяем сервисы
```
kubectl get svc nginx-service
kubectl get svc nginx-exporter-service
```
## Проверяем ServiceMonitor
```kubectl get servicemonitor -n monitoring```

## Проверяем метрики в Prometheus
```
kubectl port-forward -n monitoring svc/prometheus-prometheus-oper-prometheus 9090:9090
```