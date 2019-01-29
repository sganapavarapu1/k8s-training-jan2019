## Secrets and ConfigMaps

Objects of type secret are intended to hold sensitive information, such as passwords, OAuth tokens, and ssh keys.

Putting this information in a secret is safer and more flexible than putting it verbatim in a pod definition or in a Docker image.

### Exercise 1: Storing secrets in k8s

1. Create a file with secrets in Cloud Shell
    ```
    echo -n 'Ood7ooch8a' > ./password.txt
    ```

1. Create a secret in k8s from file
    ```
    kubectl create secret generic password --from-file=./password.txt
    ```

1. Get information about created secrets in k8s
    ```
    kubectl get secrets
    ```

1. Describe previously created secret
    ```
    kubectl describe secrets/password
    ```

1. Save the following file as `secretpod.yaml`
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: secretpod
    spec:
      containers:
      - name: secretpod
        image: redis
        volumeMounts:
        - name: pass
          mountPath: "/tmp/pass"
          readOnly: true
      volumes:
      - name: pass
        secret:
          secretName: password
    ```

1. Create the pod with attached secret password file
    ```
    kubectl create -f secretpod.yaml
    ```

1. Login to created Pod
    ```
    kubectl exec -it secretpod -- /bin/bash
    ```

1. Get password
    ```
    cat /tmp/pass/password.txt
    ```

1. Delete the pod and secret
    ```
    kubectl delete pod secretpod
    kubectl delete secret password
    ```

### Exercise 2 (Optional): Access secrets from environment variables

1. Deploy`secretpod` but expose secret using an environment variables instead of using `volumeMount`. Use [official documentation](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables) for reference.
1. Exec into the container and ensure that you can access the secret from an environment variable.

### Exercise 3 (Optional): Use config maps
1. Convert the secret from the previous exercise to ConfigMap. Use [official documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) for reference.

---

Next: [Volumes and Data](volumes.md)
