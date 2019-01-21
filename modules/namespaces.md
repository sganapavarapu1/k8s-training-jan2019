## Namespaces and RBAC

Namespaces provide for a scope of Kubernetes objects. You can think of it as a workspace you're sharing with other users. 

Many objects such as pods and services are namespaced, while some (like nodes) are not.

### Exercise 1: Use namespaces 

1. List all namespaces in the system.
    ```
    kubectl get ns
    ```

1. Use `describe` to learn more about a particular namespace.
    ```
    kubectl describe ns default
    ```

1. Create a new namespace called test 

    Save the following file as `ns.yaml`
    ```
    apiVersion: v1
    kind: Namespace
    metadata:
      name: test
    ```
    Deploy new namesapce
    ```
    kubectl create -f ns.yaml
    ```

1. Deploy a pod into the new namespace

    Save the following file as `testns-pod.yaml`
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: testns-pod
    spec:
      containers:
      - name: sise
        image: mhausenblas/simpleservice:0.5.0
        ports:
        - containerPort: 9876
    ```
    Deploy the pod
    ```
    kubectl create --namespace=test -f testns-pod.yaml
    ```
    Here we specify namespace in the command itself, though it is also posible to add the `namespace` field to the pod manifest.

1. Run the following command to list all pods in `test` namespace
    ```
    kubectl get pods --namespace=test
    ```

### Exercise 2: Use RBAC 

Usually, kubernetes is integrated with an identity provider, such as LDAP. 

In this exercise, however, we are going to manually create a service account and assign a role to it demonstrating how to use roles in kubernetes. 

1. Create user credentials. 

    Kubernetes does not have API Objects for User Accounts. 
    
    Of the available ways to manage authentication, we will use OpenSSL certificates for simplicity.

    * Create a private key for your user. In this example, we will name the file employee.key
        ```
        mkdir certs 
        cd certs 
        openssl genrsa -out employee.key 4096
        ```
    * Create a certificate sign request employee.csr using the private key you just created
        ```
        openssl req -new -key employee.key -out employee.csr -subj "/CN=employee/O=altoros"
        ```
    * Copy your Kubernetes cluster certificate authority (CA).

        ```
        gsutil cp $KOPS_STATE_STORE/simple.k8s.local/pki/private/ca/*.key ca.key
        gsutil cp $KOPS_STATE_STORE/simple.k8s.local/pki/issued/ca/*.crt ca.crt
        ```
        Note, that `KOPS_STATE_STORE` environment variable should be set to kops state store. 

    * Generate the final certificate employee.crt by approving the certificate sign request, employee.csr, you made earlier
        ```
        openssl x509 -req -in employee.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out employee.crt -days 500
        ```
    * Add a new context with the new credentials for your Kubernetes cluster.

        ```
        kubectl config set-credentials employee --client-certificate=employee.crt  --client-key=employee.key
        kubectl config set-context employee-context --cluster=simple.k8s.local --namespace=test --user=employee
        ```
    * Now you should get an access denied error when using the kubectl CLI with this configuration file. This is expected as we have not defined any permitted operations for this user

        ```
        kubectl --context=employee-context get pods
        ```

1. Create the role for managing deployments
    
    * Create a `role-deployment-manager.yaml` file with the content below
        ```
        kind: Role
        apiVersion: rbac.authorization.k8s.io/v1beta1
        metadata:
          namespace: test
          name: deployment-manager
        rules:
        - apiGroups: ["", "extensions", "apps"]
          resources: ["deployments", "replicasets", "pods"]
          verbs: ["get", "list", "watch", "create", "update", "patch", "delete"] # You can also use ["*"]
        ```

    * Create the Role in the cluster
        ```
        kubectl create -f role-deployment-manager.yaml
        ```

1. Bind the role to the employee user
    * Create a `rolebinding-deployment-manager.yaml` file with the content below.
        ```
        kind: RoleBinding
        apiVersion: rbac.authorization.k8s.io/v1beta1
        metadata:
          name: deployment-manager-binding
          namespace: test
        subjects:
        - kind: User
          name: employee
          apiGroup: ""
        roleRef:
          kind: Role
          name: deployment-manager
          apiGroup: ""
        ```

    * Deploy the RoleBinding

        ```
        kubectl create -f rolebinding-deployment-manager.yaml
        ```

1. Test The RBAC Rule

    * Now you should be able to execute the following commands without any issues
        ```
        kubectl --context=employee-context apply -f https://k8s.io/docs/tasks/run-application/deployment.yaml
        kubectl --context=employee-context get pods
        ```
    * If you run the same command with the --namespace=default argument, it will fail, as the employee user does not have access to this namespace
        ```
        kubectl --context=employee-context get pods --namespace=default
        ```

### Exercise 3 (Optional): Namespace resource limits 

1. Namespaces are commonly used with [resource quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/). Assign some quota to the test namespace and then try to use more resources then quota allows. See the link for more information how to work with quotas.

### Exercise 4 (Optional): Assign the default view cluster role to a user and try to deploy a pod 

1. There are some default user-facing roles that can be used to easily assign cluster-wide or namespace permissions. See the [reference link](https://kubernetes.io/docs/admin/authorization/rbac/#user-facing-roles) for more information.
1. Create a ClusterRoleBinding with the `view` ClusterRole and the `employee` user as a subject.
1. Try deploy a pod to the default namespace using the employee context
1. Try list the pod in the default namespace using the employee context

### Cleanup
1. Delete the namespace (which will also delete the pods, role and rolebinding)
    ```
    kubectl delete namespace test
    ```
