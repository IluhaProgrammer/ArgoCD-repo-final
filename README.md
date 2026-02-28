# 🚀 Kubernetes Manifests (GitOps Repository)

## 📌 Overview

Этот репозиторий содержит Helm chart для деплоя приложения в Kubernetes.

Развертывание происходит через **GitOps-подход** с использованием:

- Kubernetes (k3s)
    
- Helm
    
- ArgoCD
    
- HPA
    
- Ingress
    

⚡ ArgoCD отслеживает изменения в этом репозитории и автоматически синхронизирует состояние кластера.

## 🏗️ GitOps Architecture

```
Developer → Git Push → Git Repository
                              ↓
                          ArgoCD
                              ↓
                         Kubernetes
```

## 📂 Repository Structure

```
k8s-manifest/
├── charts/
│   └── app/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── namespace.yaml
│           ├── app-configmap.yaml
│           ├── app-deployment.yaml
│           ├── app-service.yaml
│           ├── app-hpa.yaml
│           ├── nginx-deployment.yaml
│           ├── nginx-service.yaml
│           ├── nginx-hpa.yaml
│           └── ingress.yaml
```

# 📦 Helm Chart

## 🔹 Chart.yaml

Содержит базовую информацию о чарте:

```
apiVersion: v2
name: myapp
description: My production app
type: application
version: 0.1.0
appVersion: "1.0.0"
```

## 🔹 values.yaml

Файл содержит конфигурацию:

```
# Эти переменные общие
  
namespace:
  name: production
  
ingress:
  name: prod-ingress-nginx
  className: nginx
  path: /api  

# Переменные для работы прложения application

app:
  name: prod-deploy-app
  replicaCount: 2
  container:
    name: prod-app
    port: 8000
  image:
    name: registry.gitlab.com/iluhaprogrammer/final-proc-app/app
    tag: latest
  resources:
    requests:
      cpu: 100m
      memory: 128mi
    limits:
      cpu: 200m
      memory: 256mi
  HPA:
    name: prod-hpa-app
    min: 2
    max: 5
    type: Resources
    target:
      type: Utilization
      avg: 50
  service:
    name: prod-service-app
    type: ClusterIP
    port: 80
  label:
    first: app
    second: prod
  configMap:
    name: prod-config-app
    values:
      APP_NAME: prod-app
      APP_STAGE: production
      LOG_LEVEL: info
      DEBUG: false

# Переменные для nginx

nginx:
  name: prod-deploy-nginx
  replicaCount: 1
  container:
    name: prod-nginx
    port: 80
  image:
    name: registry.gitlab.com/iluhaprogrammer/final-proc-app/nginx
    policy: IfNotPresent
    tag: latest
  resources:
    requests:
      cpu: 100m
      memory: 128mi
    limits:
      cpu: 200m
      memory: 256mi
  HPA:
    name: prod-hpa-nginx
    min: 1
    max: 3
    type: Resources
    target:
      type: Utilization
      avg: 50
  service:
    name: prod-service-nginx
    type: ClusterIP
    port: 90
  label:
    first: nginx
    second: prod
```

⚡ Все шаблоны используют `.Values` для гибкости.

# 🧩 Kubernetes Components

## 🏷 Namespace

Создает изолированное пространство:

```
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Values.namespace.name }}
```

## 🚀 Application Deployment

- ReplicaSet
    
- RollingUpdate strategy
    
- Resource limits
    
- ConfigMap
    
- Environment variables

## 🌐 Service

Тип: `ClusterIP`

Обеспечивает внутреннюю коммуникацию.

## 📈 HPA (Horizontal Pod Autoscaler)

- Метрика: CPU
    
- minReplicas: 2
    
- maxReplicas: 5
    

Позволяет масштабировать приложение автоматически.


## 🧱 Nginx Deployment

Используется как reverse proxy перед приложением.

## 🌍 Ingress

Настраивает внешний доступ к сервису через:

- Host-based routing
    
- TLS (если используется)
    
- Path-based routing

# 🔥 ArgoCD Integration

ArgoCD настроен на:

- Отслеживание branch (например, `main`)
    
- Автоматическую синхронизацию
    
- Self-healing
    
- Prune удалённых ресурсов
    

Пример Application:

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app-release
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://github.com/IluhaProgrammer/ArgoCD-repo-final.git
    targetRevision: main
    path: charts/app
    helm:
      valueFiles:
        - values.yml
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

# 🚀 Deployment Flow

1. Изменение `values.yaml`
    
2. Commit → Push
    
3. ArgoCD обнаруживает изменения
    
4. Синхронизация
    
5. Kubernetes приводит состояние к desired state

## Что дальше?

Это в принципе финальный шаг, который можно было сделать, тут реализован репо по хранению helm charts для деплоя нашего приложения на кубер

### Что можно улучшить?

Что можно было бы добавить в проект?
- Postgres + replication + backup
- HashiCorp Vault для хранения секретов
- Redis
- Prometheus + Grafana + AlertManager
- ELK стэк
- Kafka
- Сделать микросервисную ахритектуру

## Автор:

@Hasler4444 - telegram
iluharog@gmail.com - почта для связи
