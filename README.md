## Add your registry to k8s
1. Check your docker config
    ```bash
    cat ~.docker/config.json
    ```

2. add docker secret to k8s
    ```bash
    kubectl create secret generic regcred -n argocd \
        --from-file=~/.dockerconfigjson=/home/${USER}docker/config.json \
        --type=kubernetes.io/dockerconfigjson
    ```
