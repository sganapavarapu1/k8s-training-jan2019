## Services

A service is an abstraction for pods, providing a stable, virtual IP (VIP) address.

While pods may come and go, services allow clients to reliably connect to the containers running in the pods, using the VIP. The virtual in VIP means it’s not an actual IP address connected to a network interface but its purpose is purely to forward traffic to one or more pods.

Keeping the mapping between the VIP and the pods up-to-date is the job of kube-proxy, a process that runs on every node, which queries the API server to learn about new services in the cluster.

### Exercise 1: Deploying PHP Guestbook application with Redis

1. Save the following file as `redis-master-deployment.yaml`
    ```
    apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
    kind: Deployment
    metadata:
      name: redis-master
    spec:
      selector:
        matchLabels:
          app: redis
          role: master
          tier: backend
      replicas: 1
      template:
        metadata:
          labels:
            app: redis
            role: master
            tier: backend
        spec:
          containers:
          - name: master
            image: k8s.gcr.io/redis:e2e
            resources:
              requests:
                cpu: 100m
                memory: 100Mi
            ports:
            - containerPort: 6379
    ```

1. Create the Redis Master Deployment
    ```
    kubectl apply -f redis-master-deployment.yaml
    ```

1. Query the list of Pods to verify that the Redis Master Pod is running.
    ```
    kubectl get pods
    ```

1. Save the following file as `redis-master-service.yaml`
    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: redis-master
      labels:
        app: redis
        role: master
        tier: backend
    spec:
      ports:
      - port: 6379
        targetPort: 6379
      selector:
        app: redis
        role: master
        tier: backend
    ```
    Pay attention to `selector` and `ports` fields. Make sure you understand how service is connected to deployment.

1. Deploy the service.
    ```
    kubectl apply -f redis-master-service.yaml
    ```

1. Save the following file as `redis-slave-deployment.yaml`
    ```
    apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
    kind: Deployment
    metadata:
      name: redis-slave
    spec:
      selector:
        matchLabels:
          app: redis
          role: slave
          tier: backend
      replicas: 2
      template:
        metadata:
          labels:
            app: redis
            role: slave
            tier: backend
        spec:
          containers:
          - name: slave
            image: gcr.io/google_samples/gb-redisslave:v1
            resources:
              requests:
                cpu: 100m
                memory: 100Mi
            env:
            - name: GET_HOSTS_FROM
              value: dns
            ports:
            - containerPort: 6379
    ```

1. Apply the redis slave deployment.
    ```
    kubectl apply -f redis-slave-deployment.yaml
    ```

1. Save the following file as `redis-slave-service.yaml`
    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: redis-slave
      labels:
        app: redis
        role: slave
        tier: backend
    spec:
      ports:
      - port: 6379
      selector:
        app: redis
        role: slave
        tier: backend
    ```

1. Deploy the redis slave service.
    ```
    kubectl apply -f redis-slave-service.yaml
    ```

1. Save the following file as `frontend-deployment.yaml`
    ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: frontend
    spec:
      selector:
        matchLabels:
          app: guestbook
          tier: frontend
      replicas: 3
      template:
        metadata:
          labels:
            app: guestbook
            tier: frontend
        spec:
          containers:
          - name: php-redis
            image: gcr.io/google-samples/gb-frontend:v4
            resources:
              requests:
                cpu: 100m
                memory: 100Mi
            env:
            - name: GET_HOSTS_FROM
              value: dns
            ports:
            - containerPort: 80
    ```

1. Apply the frontend deployment.
    ```
    kubectl apply -f frontend-deployment.yaml
    ```

1. Save the following file as `frontend-service.yaml`
    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: frontend
      labels:
        app: guestbook
        tier: frontend
    spec:
      type: LoadBalancer
      ports:
      - port: 80
      selector:
        app: guestbook
        tier: frontend
    ```
    Pay attension to the service type.

1. Deploy the fronted service.
    ```
    kubectl apply -f frontend-service.yaml
    ```

1. Run `kubectl get services` to list all services.

1. Run the following command to get the IP address for the frontend Service.
    ```
    kubectl get service frontend
    ```

1. In GCE Cloud Console, find and investigate the external IP address that the `LoadBalancer` service type created
    * VPC Network -> External IP addresses

1. Copy the External IP address, and load the page in your browser to view the application.

1. Run the following command to scale up the number of frontend Pods
    ```
    kubectl scale deployment frontend --replicas=5
    ```

### Exercise 2 (Optional): Investigate source code of the sample

1. The source code of the previously deployed sample can be found [here](https://github.com/kubernetes/examples/tree/master/guestbook)
1. The files we are interested in:
    * Redis slave Dockerfile: [link](https://github.com/kubernetes/examples/blob/master/guestbook/redis-slave/Dockerfile)
    * Redis slave startup script: [link](https://github.com/kubernetes/examples/blob/master/guestbook/redis-slave/run.sh)
    * PHP application: [link](https://github.com/kubernetes/examples/blob/master/guestbook/php-redis/guestbook.php)
1. Make sure you understand the following:
    * How a redis slave connects to the redis master? What address is it using?
    * How the php app connects to both the redis master and redis slaves?

### Exercise 3 (Optional): Manually connect to redis from app pod

1. Go inside any frontend pods.
1. Use `redis-tools` package to install [redis-cli](https://redis.io/topics/rediscli)
1. Use `redis-cli` to connect to redis master.

### Exercise 4 (Optional): Blue green deployment

1. Create a deployment called "blue"  with label "app=blue".
1. Create a service that uses the same selector "app=blue"  
1. Create a second deployment with label "app=green". The deployment should contain the same application. (in a real scenario this should be a different version of the app, but for your example, you can use exactly the same app)
1. Change service selector to "app=green" and make sure that now the service switched to the second deployment.

### Cleanup

1. Delete the services and deployments
    ```
    kubectl delete service frontend redis-slave redis-master
    kubectl delete deployment frontend redis-slave redis-master
    ```

---

Next: [Secrets and ConfigMaps](secrets_and_config_maps.md)
