## Add your docker registry to k8s
1. Check your docker config
    ```bash
    cat ~.docker/config.json
    ```

2. add docker secret to k8s
    ```bash
    kubectl create secret generic argocd-docker-secret -n argocd \
    --from-file=.dockerconfigjson=.docker/config.json \ 
    --type=kubernetes.io/dockerconfigjson
    ```

## Add custom registry
```bash
kubectl create secret docker-registry nexus-secret -n argocd \
--docker-server=https://docker.home.local \
--docker-username=admin \
--docker-password='!QAZ2wsx' 
```
