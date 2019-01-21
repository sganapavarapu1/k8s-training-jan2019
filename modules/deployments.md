## Deployments

A deployment is a supervisor for pods and replica sets, giving you fine-grained control over how and when a new pod version is rolled out as well as rolled back to a previous state.

### Exercise 1: Create a deployment 

1. Save the following file as `deployment.yaml`.
    ```console
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: simpleservice
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: simpleservice
      template:
        metadata:
          labels:
            app: simpleservice
        spec:
          containers:
          - name: simpleservice
            image: mhausenblas/simpleservice:0.5.0
            ports:
            - containerPort: 9876
            env:
            - name: SIMPLE_SERVICE_VERSION
              value: "0.9"
    ```

1. Create deployment.
    ```
    kubectl create -f deployment.yaml
    ```

1. Check deployment, replica set and pods, created by the previous command.
    ```
    kubectl get deploy
    kubectl get rs
    kubectl get pods
    ```
    Copy pod IP address 

1. SSH to any kubernetes node and query app info.
    ```
    curl <pod-ip>:9876/info
    ``` 
    Make sure that simpleservice returns version `0.9`

1. Update `deployment.yaml` and set `SIMPLE_SERVICE_VERSION` to `1.0`.

1. Apply changes.
    ```
    kubectl apply -f deployment.yaml
    ``` 
1. Run 
    ```
    kubectl get pods
    ```
    What we now see is the rollout of two new pods with the updated version 1.0 as well as the two old pods with version 0.9 being terminated.
    ```
    NAME                                 READY     STATUS        RESTARTS   AGE
    simpleservice-5f5fb45496-6gk2j   1/1       Terminating   0          17m
    simpleservice-5f5fb45496-v5czm   1/1       Terminating   0          17m
    simpleservice-84f7f575d8-pvwcq   1/1       Running       0          15s
    simpleservice-84f7f575d8-qt254   1/1       Running       0          17s
    ```

1. Make sure that new replica set has been created.
    ```
    kubectl get rs
    ```

1. Once again get pod ip, ssh to one of the nodes and send a request to simpleservice.
    ```
    curl <pod-ip>:9876/info
    ```
    Make sure that version now is "1.0"

1. Check deployment history.
    ```
    kubectl rollout history deploy/simpleservice
    ```

1. If there are problems in the deployment Kubernetes will automatically roll back to the previous version, however you can also explicitly roll back to a specific revision, as in our case to revision 1 (the original pod version).
    ```
    kubectl rollout undo deploy/simpleservice --to-revision=1
    ```
    At this point in time we're back at where we started, with two new pods serving again version 0.9.

### Exercise 2 (Optional): Observe how kubernetes restarts containers 

1. Use the simpleservice deployment
1. Exec into the container, find and kill web server process
1. Observe whether kubernetes tries to redeploy container

### Clean-up 

1. Delete the deployment.
    ```
    kubectl delete deployment simpleservice
    ```
