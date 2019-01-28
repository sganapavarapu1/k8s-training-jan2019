## Health Checks

### Exercise 1: Deploy a pod with a health check

1. Create a pod that exposes an endpoint /health, responding with an HTTP 200 status code

   Save the following file as `hc.yaml`
    ```
    cat > hc.yaml <<EOF
    apiVersion: v1
    kind: Pod
    metadata:
      name: hc
    spec:
      containers:
      - name: simpleservice
        image: mhausenblas/simpleservice:0.5.0
        ports:
        - containerPort: 9876
        livenessProbe:
          initialDelaySeconds: 2
          periodSeconds: 5
          httpGet:
            path: /health
            port: 9876
    EOF  
    ```
    Pay attention to `livenessProbe` section.

1. Deploy the pod
    ```
    kubectl create -f hc.yaml
    ```

1. Describe the pod; it should be considered healthy
    ```
    kubectl describe pod hc
    ```

1. Now we launch a bad pod, that is, a pod that has a container that randomly (in the time range 1 to 4 sec) does not return a 200 code
    Save the following file as `bad-hc.yaml`
    ```
    cat > bad-hc.yaml <<EOF
    apiVersion: v1
    kind: Pod
    metadata:
      name: bad-hc
    spec:
      containers:
      - name: simpleservice
        image: mhausenblas/simpleservice:0.5.0
        ports:
        - containerPort: 9876
        env:
        - name: HEALTH_MIN
          value: "1000"
        - name: HEALTH_MAX
          value: "4000"
        livenessProbe:
          initialDelaySeconds: 2
          periodSeconds: 5
          httpGet:
            path: /health
            port: 9876
    EOF
    ```

1. Deploy the pod
    ```
    kubectl create -f bad-hc.yaml
    ```

1. Check events at the bad pod
    ```
    kubectl describe pod bad-hc
    ```
    You should see that bad pod was restarted several times.

1. Delete the pods
    ```
    kubectl delete pod hc bad-hc
    ```    


### Exercise 2: Use readiness probe

1. Create a pod `readiness.yaml` with a readinessProbe that kicks in after 10 seconds
    ```
    cat > readiness.yaml <<EOF
    apiVersion: v1
    kind: Pod
    metadata:
      name: readiness
    spec:
      containers:
      - name: simpleservice
        image: mhausenblas/simpleservice:0.5.0
        ports:
        - containerPort: 9876
        readinessProbe:
          initialDelaySeconds: 10
          httpGet:
            path: /health
            port: 9876
    EOF
    ```

1. Deploy the pod
    ```
    kubectl create -f readiness.yaml
    ```

1. Looking at the events of the pod.
    You should see that, eventually, the pod is ready to serve traffic
    ```
    kubectl describe pod readiness
    ```

1. Delete the pod
    ```
    kubectl delete pod readiness
    ```

### Exercise 3 (Optional): Create health check for nginx pod

1. Deploy a pod that runs nginx and uses port 80 and root path for the health check. Ensure that pod is healthy.
1. Change health check configuration to make pod unhealthy.
1. Observe whether kubernetes tries to restart the unhealthy pod.

### Exercise 4 (Optional): Create health check using TCP sockets

1. Modify `hc.yaml` to use a TCP socket handler instead of HTTP GET handler. [Reference link](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.10/#handler-v1-core)
1. Deploy the pod and see if the pod is considered healthy.

---

Next: [Deployments](deployments.md)
