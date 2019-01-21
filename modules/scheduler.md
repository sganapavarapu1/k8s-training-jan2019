## Scheduler

### Exercise 1:  Disabling the scheduler

1. SSH to the master node.

1. Move the kube-scheduler manifest out of the `/etc/kubernetes/manifests/` folder.
    ```
    sudo mv /etc/kubernetes/manifests/kube-scheduler.manifest ~
    ```
1. Wait until the kubelet shuts down the scheduler pod. This can be checked by listing all system pods.
    ```
    kubectl --namespace kube-system get pods
    ```

1. Deploy a pod normally.

1. Use the `get pods` command to list all pods. The one you've just deployed should be in a pending state with no node assigned to it.

1. Use the `describe pod` command to check which node is assigned to the pod.

1. Return the kube-scheduler manifest back to the `/etc/kubernetes/manifests/` folder on the master node.
    ```
    sudo mv ~/kube-scheduler.manifest /etc/kubernetes/manifests/
    ```

1. Wait until the scheduler runs and make sure that a node is now assigned to the pod and the pod is running.


### Exercise 2: Manually schedule a pod 

1. While the default scheduler is disabled and a pod in the Pending state, try to manually assign a node to the container using API.
    * Use curl to sent a POST request to the `/api/v1/namespaces/{namespace}/bindings` endpoint. 
    * The body of the request should have the following format `{"apiVersion":"v1", "kind": "Binding", "metadata": {"name": "<pod-name>"}, "target": {"apiVersion": "v1", "kind": "Node", "name": "<node-name>"}}`
    * Use the official [Reference documentation](https://kubernetes.io/docs/reference/) and correct version of the [API Reference](https://v1-9.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#binding-v1-core) to help.  
    ```
    curl -X POST 127.0.0.1:8080/api/v1/namespaces/default/bindings -H "Content-Type:application/json" -d '{"apiVersion":"v1", "kind": "Binding", "metadata": {"name": "twocontainers"}, "target": {"apiVersion": "v1", "kind": "Node", "name": "master-us-west1-c-4lmf"}}'
    ```
    * Make sure you copy/move the kube-scheduler manifest back into `/etc/kubernetes/manifests/`


