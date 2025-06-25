## Обзор
Этот репозиторий содержит конфигурации для развертывания приложения Petclinic с использованием:
- **ArgoCD** для управления развертываниями
- **ArgoCD Image Updater** для автоматического обновления образов
- **Kustomize** для управления конфигурациями Kubernetes

## Структура файлов
```
petclinic-app/
├── 01-petclinic-dep.yaml      # Манифест Deployment
├── 02-petclinic-svc.yaml      # Манифест Service
├── 03-nexus-secret.yaml       # Secret для доступа к Docker-репозиторию
└── kustomization.yaml         # Конфигурация Kustomize
argocd-app.yaml                # Конфигурация приложения ArgoCD
```

## Конфигурация компонентов

### 1. Deployment (01-petclinic-dep.yaml)
- Реплики: 1
- Использует образ: `registry.home-local.site/petclinic`
- Пробники:
  - Readiness: HTTP GET на порт 8080, путь /
  - Liveness: HTTP GET на порт 8080, путь /
- Порт контейнера: 8080

### 2. Service (02-petclinic-svc.yaml)
- Тип: ClusterIP
- Порт: 8080 → 8080 (targetPort)

### 3. Secret (03-nexus-secret.yaml)
- Тип: `kubernetes.io/dockerconfigjson`
- Содержит учетные данные для доступа к Docker-репозиторию

### 4. Kustomize (kustomization.yaml)
- Объединяет все ресурсы
- Управляет тегом образа (текущая версия: 1.0.0-439)

### 5. ArgoCD Application (argocd-app.yaml)
- **Источник**: GitHub репозиторий `vladoz77/petclinic-cd`
- **Путь**: `petclinic-app`
- **Назначение**: namespace `petclinic` (создается автоматически)
- **Синхронизация**: автоматическая с самовосстановлением

## Настройка ArgoCD Image Updater
```yaml
annotations:
  argocd-image-updater.argoproj.io/image-list: petclinic-app=registry.home-local.site/petclinic
  argocd-image-updater.argoproj.io/petclinic-app.update-strategy: latest
  argocd-image-updater.argoproj.io/petclinic-app.ignore-tags: latest
  argocd-image-updater.argoproj.io/write-back-method: git
  argocd-image-updater.argoproj.io/write-back-target: kustomization
  argocd-image-updater.argoproj.io/git-branch: main
  argocd-image-updater.argoproj.io/petclinic-app.force-update: "true"
```

### Параметры Image Updater:
1. **Образ для отслеживания**: `registry.home-local.site/petclinic`
2. **Стратегия обновления**: `latest` (использует последний доступный тег)
3. **Игнорируемые теги**: `latest`
4. **Метод записи изменений**: Git (обновляет kustomization.yaml)
5. **Ветка Git**: `main`
6. **Принудительное обновление**: включено

## Развертывание

### Предварительные требования
1. Установленный и настроенный ArgoCD в кластере
2. Установленный ArgoCD Image Updater
3. Доступ к Docker-репозиторию `registry.home-local.site`
4. Настроенные Secrets для доступа к репозиторию

### Шаги развертывания
1. Применить Application в ArgoCD:
   ```bash
   kubectl apply -f argocd-app.yaml -n argocd
   ```
2. ArgoCD автоматически:
   - Создаст namespace `petclinic`
   - Развернет приложение
   - Начнет отслеживать обновления образов

