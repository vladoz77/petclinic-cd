**Создаем секрет `nexus-secret` для доступа в реестр контейнеров** 

```bash
kubectl create secret docker-registry nexus-secret -n argocd \
--docker-server=https://registry.home-local.site
--docker-username='argocd'\
--docker-password='!QAZ2wsx'
```
Где:
- `harbor-secret`: имя секрета
- `--docker-server`: адрес реестра образов
- `--docker-username`: имя пользователя реестра
- `--docker-password`: пароль реестра

 
**Создадим `values.yaml`**
```yaml
replicaCount: 1
fullnameOverride: argocd-image-updater
config:
  registries:
    - name: registry.home-local.site
      prefix: registry.home-local.site
      api_url: https://registry.home-local.site
      credentials: pullsecret:argocd/nexusr-secret
metrics:
  enabled: true
  service:
      annotations:
          prometheus.io/scrape: "true"
          prometheus.io/port: "8081"
```

**Установка argocd image updater**
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd-image-updater argo/argocd-image-updater -f values.yaml -n argocd
```
