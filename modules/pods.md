## Pods

A pod is a collection of containers sharing a network and mount namespace and is the basic unit of deployment in Kubernetes. All containers in a pod are scheduled on the same node.

### Exercise 1: Launch a pod using the container image

1. Run a pod.
    ```
    kubectl run simpleservice --image=mhausenblas/simpleservice:0.5.0 --port=9876
    ```

1. List pods and ensure that out pod is running and copy pod name.
    ```
    kubectl get pods
    ```

1. Describe pod and copy its IP address.
    ```
    kubectl describe pod <pod-name>
    ```

1. From GCP console VM instances tab ssh to any of the nodes, either worker or master. (GCP allows you to ssh using web browser just by clicking on SSH button or with a new Cloud Shell session) This step is required because by default pod network is not open to outside world.

    ```
    gcloud compute instances list

    NAME                    ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP      STATUS
    master-us-west1-c-hkkn  us-west1-c  n1-standard-1               10.138.0.21  104.199.123.250  RUNNING
    nodes-4vdd              us-west1-c  n1-standard-2               10.138.0.23  35.233.214.200   RUNNING
    nodes-jl86              us-west1-c  n1-standard-2               10.138.0.22  35.247.98.64     RUNNING
    ```

    Choose one of the systems from your list to log into.
    ```
    gcloud compute --project "$DEVSHELL_PROJECT_ID" ssh --zone "us-west1-c" "<vm-instance-name>"
    ```

1. Connect to the pod.
    ```
    curl <pod-ip>:9876/info
    ```
    The output should look like this.
    ```
    {"host": "100.96.2.4:9876", "version": "0.5.0", "from": "10.128.0.4"}
    ```
    This is a response send by the application running inside the pod.

1. `kubectl run` creates a deployment, so in order to get rid of the pod you have to execute the following command.
    ```
    kubectl delete deployment simpleservice
    ```

### Exercise 2: Launch a pod using the configuration file

1. Save the following file as `pod.yaml`
    ```console
    cat > pod.yaml <<EOF
    apiVersion: v1
    kind: Pod
    metadata:
      name: twocontainers
    spec:
      containers:
      - name: simpleservice
        image: mhausenblas/simpleservice:0.5.0
        ports:
        - containerPort: 9876
      - name: shell
        image: centos:7
        command:
          - "bin/bash"
          - "-c"
          - "sleep 10000"
    EOF
    ```
    Here we specify that our new pod should contain 2 containers. The first one runs the same application as previously. The second one runs sleep command.

1. Create a pod from `pod.yaml` configuration file.
    ```
    kubectl create -f pod.yaml
    ```

1. Navigate inside the second container.
    ```
    kubectl exec twocontainers -c shell -i -t -- bash
    ```

1. Access the simpleservice on localhost.
    ```
    curl -s localhost:9876/info
    ```

1. Delete the pod.
    ```
    kubectl delete pod twocontainers
    ```

### Exercise 3 (Optional): Deploy a pod from custom image.

1. Push `nginx` image, created in the docker part of the course, to dockerhub. [Reference link](https://ropenscilabs.github.io/r-docker-tutorial/04-Dockerhub.html)
1. Run a pod using this image.

### Exercise 4 (Optional): Limit pod resources

1. Set [resources](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/) property to limit how much memory and CPU the pod can use.
1. Use [stress](https://linux.die.net/man/1/stress) to load the container, see what happens.

---

Next: [Health Checks](health.md)
