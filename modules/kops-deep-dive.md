## kops deep dive

### Exercise 1: Installing the Kubernetes Dashboard Addon

1. Install the [dashboard](https://github.com/kubernetes/dashboard) addon.
    ```
    kubectl create -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/kubernetes-dashboard/v1.8.3.yaml
    ```

1. Create a `dashboard-service-account.yaml` manifest with a service account and cluster role binding.
    ```
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: admin-user
      namespace: kube-system

    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: admin-user
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: admin-user
      namespace: kube-system
    ```

1. Create and bind the role.
    ```
    kubectl create -f dashboard-service-account.yaml
    ```

1. Get the admin-user secret token.
    ```
    kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
    ```

1. Start a proxy on port 8080.
    ```
    kubectl proxy -p 8080
    ```

1. Forward port 8080 from the Cloud Shell to your local machine.

    From Cloud Shell top bar select option `Preview on port 8080`

1. View the dashboard service on the /proxy/ endpoint.

    Append `api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/` to the top level domain.

    E.g., `https://8080-dot-3438793-dot-devshell.appspot.com/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/`

1. From the dashboard UI, paste in the token value. Ensure there are no line breaks when copying the token.

1. You should be successfully logged in as admin-user.

1. Explore the Kubernetes Dashboard and what information you can find there.

### Exercise 2: Using kops to change the cluster configuration

1. Run the following command to edit cluster configuration.
    ```
    kops edit cluster
    ```

1. Find `kubernetesVersion` and upgrade Kubernetes to a newer patch version.
    ```
      kubernetesVersion: 1.9.7
    ```

1. Apply the changes.
    ```
    export KOPS_FEATURE_FLAGS=AlphaAllowGCE # If unset
    kops update cluster # to preview
    kops update cluster --yes # to apply
    kops rolling-update cluster # to preview the rolling-update
    kops rolling-update cluster --yes # to roll all your instances
    ```
    Pay attention to how kops drains all pods from the node that is being updated. This allows kops to make the update without app downtime.

1. Check that the server version of kubernetes has been upgraded successfully
    ```
    kubectl version
    kubectl get nodes
    ```

### Exercise 3 (Optional): Deploy a Highly Available Cluster

1. Deploy a new cluster.  Follow the instructions in the [kops documentation](https://github.com/kubernetes/kops/blob/master/docs/high_availability.md)
1. Delete the second cluster.

### Exercise 3 (Optional): Add Heapster metrics to the Kubernetes Dashboard

1. Deploy Heapster inside the `kube-system` namespace and expose a `heapster` service. [Reference link](https://github.com/kubernetes/dashboard/wiki/Integrations)
1. Modify the dashboard deployment to use heapster
