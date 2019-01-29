## Stateful Sets

StatefulSet is the workload API object used to manage stateful applications.

Like a Deployment, a StatefulSet manages Pods that are based on an identical container spec.

Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

### Exercise 1: Deploying Cassandra with Stateful Sets

1. Create a Cassandra Headless Service

    Save the following file as `cassandra-service.yaml`
    ```
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        app: cassandra
      name: cassandra
      annotations:
        service.alpha.kubernetes.io/tolerate-unready-endpoints: "true"
    spec:
      clusterIP: None
      ports:
      - port: 9042
      selector:
        app: cassandra
    ```
    This Service is used for DNS lookups between Cassandra Pods and clients within the Kubernetes Cluster. Pay attension to `service.alpha.kubernetes.io/tolerate-unready-endpoints: "true"` field - without it the first pod will not be exposed untill its readiness probe completes.

1. Deploy the service
    ```
    kubectl create -f cassandra-service.yaml
    ```

1. Use a StatefulSet to Create a Cassandra Ring

    Save the following file as `cassandra-statefulset.yaml`
    ```
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: cassandra
      labels:
        app: cassandra
    spec:
      serviceName: cassandra
      replicas: 3
      selector:
        matchLabels:
          app: cassandra
      template:
        metadata:
          labels:
            app: cassandra
        spec:
          terminationGracePeriodSeconds: 1800
          containers:
          - name: cassandra
            image: gcr.io/google-samples/cassandra:v13
            imagePullPolicy: Always
            ports:
            - containerPort: 7000
              name: intra-node
            - containerPort: 7001
              name: tls-intra-node
            - containerPort: 7199
              name: jmx
            - containerPort: 9042
              name: cql
            resources:
              limits:
                cpu: "500m"
                memory: 1Gi
              requests:
               cpu: "500m"
               memory: 1Gi
            securityContext:
              capabilities:
                add:
                  - IPC_LOCK
            lifecycle:
              preStop:
                exec:
                  command:
                  - /bin/sh
                  - -c
                  - nodetool drain
            env:
              - name: MAX_HEAP_SIZE
                value: 512M
              - name: HEAP_NEWSIZE
                value: 100M
              - name: CASSANDRA_SEEDS
                value: "cassandra-0.cassandra"
              - name: CASSANDRA_CLUSTER_NAME
                value: "K8Demo"
              - name: CASSANDRA_DC
                value: "DC1-K8Demo"
              - name: CASSANDRA_RACK
                value: "Rack1-K8Demo"
              - name: POD_IP
                valueFrom:
                  fieldRef:
                    fieldPath: status.podIP
            readinessProbe:
              exec:
                command:
                - /bin/bash
                - -c
                - /ready-probe.sh
              initialDelaySeconds: 15
              timeoutSeconds: 5
            volumeMounts:
            - name: cassandra-data
              mountPath: /cassandra_data
      volumeClaimTemplates:
      - metadata:
          name: cassandra-data
        spec:
          accessModes: [ "ReadWriteOnce" ]
          resources:
            requests:
              storage: 1Gi
    ```
    Pay attension to CASSANDRA_SEEDS environment variable - nodes use it to discover each other. Make sure you understand how this variable is related to casandra service. Also take a look at `readinessProbe` - it might delay container start.

1. Deploy casandra statefull set
    ```
    kubectl create -f cassandra-statefulset.yaml
    ```

1. Get the Pods to see the ordered creation status:
    ```
    kubectl get pods -l="app=cassandra"
    ```
    The response should be...
    ```
       NAME          READY     STATUS              RESTARTS   AGE
       cassandra-0   0/1       ContainerCreating   0          3s
    ```
    Eventually the response will be...
    ```
       NAME          READY     STATUS              RESTARTS   AGE
       cassandra-0   1/1       Running             0          4m
       cassandra-1   1/1       Running             0          3m
       cassandra-2   1/1       Running             0          2m
    ```

1. Run the Cassandra utility nodetool to display the status of the ring.
    ```
    kubectl exec cassandra-0 -- nodetool status
    ```
    The response is
    ```
       Datacenter: DC1-K8Demo
       ======================
       Status=Up/Down
       |/ State=Normal/Leaving/Joining/Moving
       --  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
       UN  172.17.0.5  83.57 KiB  32           74.0%             e2dd09e6-d9d3-477e-96c5-45094c08db0f  Rack1-K8Demo
       UN  172.17.0.4  101.04 KiB 32           58.8%             f89d6835-3a42-4419-92b3-0e62cae1479c  Rack1-K8Demo
       UN  172.17.0.6  84.74 KiB  32           67.1%             a6a1e8c2-3dc5-4417-b1a0-26507af2aaad  Rack1-K8Demo
    ```  

### Exercise 2 (Optional): Scale

1. Scale cassandra cluster up and down. Observe in what order kubernetes deploys/deletes pods.

### Cleanup

1. Delete the stateful set and service
    ```
    kubectl delete pods -l="app=cassandra"
    kubectl delete service cassandra
    ```

---

Next: [Ingress](ingress.md)
