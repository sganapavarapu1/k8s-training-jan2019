## Helm

The package manager for Kubernetes

### Exercise 1: Use helm to deploy prometheus

1. Install helm
    ```
    wget https://storage.googleapis.com/kubernetes-helm/helm-v2.9.1-linux-amd64.tar.gz
    tar -xvf helm-v2.9.1-linux-amd64.tar.gz
    mv linux-amd64/helm ~/bin/
    rm helm-v2.9.1-linux-amd64.tar.gz linux-amd64/ -rf
    ```

1. Initialize helm
    ```
    helm init
    ```  
1. Create helm service account
    ```
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: tiller
      namespace: kube-system
    ---
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: tiller-clusterrolebinding
    subjects:
    - kind: ServiceAccount
      name: tiller
      namespace: kube-system
    roleRef:
      kind: ClusterRole
      name: cluster-admin
      apiGroup: ""
    ```

1. Update heml service account reference
    ```
    kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
    ```

1. Add coreos chart repository
    ```
    helm repo add coreos https://s3-eu-west-1.amazonaws.com/coreos-charts/stable/
    ```

1. Install prometheus
    ```
    helm install coreos/prometheus-operator --name prometheus-operator --set rbac.create=true
    helm install coreos/kube-prometheus --name kube-prometheus --set global.rbacEnable=true 
    ```

1. List all pods, make sure all are healthy
    ```
    kubectl get pods
    ```
1. Expose prometheus on port 9090
    ```
    kubectl port-forward $(kubectl get pods --selector=app=prometheus --output=jsonpath="{.items..metadata.name}")  9090
    ```

1. Open web preview on port 9090 and check the prometheus interface.

1. Expose grafana on port 3000
    ```
    kubectl port-forward $(kubectl get pods --selector=app=kube-prometheus-grafana --output=jsonpath="{.items..metadata.name}")  3000
    ```

1. Open web preview on port 3000 and check the grafana interface.

1. Click on `Home` at the top left corner to list all available dashboards.


### Exercise 2 (Optional): Use `helm -h` to figure out how to do the following

1. List all available charts.
1. List all deployed charts.
1. Provide configuration parameters for a chart during deployment.
1. Upgrade and rollback a chart.
1. Inspect a chart.

Try all operations, mentioned above.
