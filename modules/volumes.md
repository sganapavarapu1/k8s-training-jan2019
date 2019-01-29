## Volumes and Data

### Exercise 1: Deploying WordPress and MySQL with Persistent Volumes

1. Run the following command to make sure that your kubernetes installation has default storage class
    ```
    kubectl get storageclass
    ```
    For GCE default storage class should have `kubernetes.io/gce-pd` provisioner. This provisioner creates GCE persistent disks for any requested persistent volume.

1. Create a Secret for MySQL Password
    ```
    kubectl create secret generic mysql-pass --from-literal=password=YOUR_PASSWORD
    ```
    Replace YOUR_PASSWORD with the password you want to apply.

1. Verify that the Secret exists by running the following command
    ```
    kubectl get secrets
    ```

1. Create empty `mysql.yaml` file.

1. Add Persistent Volume Claim definition to `mysql.yaml`
    ```
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mysql-pv-claim
      labels:
        app: wordpress
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 20Gi
    ```

1. Add mysql deployment definition to `mysql.yaml`
    ```
    ---
    apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
    kind: Deployment
    metadata:
      name: wordpress-mysql
      labels:
        app: wordpress
    spec:
      selector:
        matchLabels:
          app: wordpress
          tier: mysql
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app: wordpress
            tier: mysql
        spec:
          containers:
          - image: mysql:5.6
            name: mysql
            env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-pass
                  key: password
            ports:
            - containerPort: 3306
              name: mysql
            volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql
          volumes:
          - name: mysql-persistent-storage
            persistentVolumeClaim:
              claimName: mysql-pv-claim
    ````
    Pay attension to `volumes`, `volumeMounts` and `env` fields.

1. Add service definition to `mysql.yaml`
    ```
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: wordpress-mysql
      labels:
        app: wordpress
    spec:
      ports:
        - port: 3306
      selector:
        app: wordpress
        tier: mysql
      clusterIP: None
    ```

1. Deploy MySQL from the `mysql.yaml` file
    ```
    kubectl create -f mysql.yaml
    ```

1. Verify that a PersistentVolume got dynamically provisioned
    ```
    kubectl get pvc
    ```
    It can take up to a few minutes for the PVs to be provisioned and bound.

1. Verify that the Pod is running by running the following command
    ```
    kubectl get pods
    ```

1. Create `wordpress.yaml` file with the following content
    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: wordpress
      labels:
        app: wordpress
    spec:
      ports:
        - port: 80
      selector:
        app: wordpress
        tier: frontend
      type: LoadBalancer
    ---
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: wp-pv-claim
      labels:
        app: wordpress
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 20Gi
    ---
    apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
    kind: Deployment
    metadata:
      name: wordpress
      labels:
        app: wordpress
    spec:
      selector:
        matchLabels:
          app: wordpress
          tier: frontend
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app: wordpress
            tier: frontend
        spec:
          containers:
          - image: wordpress:4.8-apache
            name: wordpress
            env:
            - name: WORDPRESS_DB_HOST
              value: wordpress-mysql
            - name: WORDPRESS_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-pass
                  key: password
            ports:
            - containerPort: 80
              name: wordpress
            volumeMounts:
            - name: wordpress-persistent-storage
              mountPath: /var/www/html
          volumes:
          - name: wordpress-persistent-storage
            persistentVolumeClaim:
              claimName: wp-pv-claim
    ```
    Pay attention to wordpress service definition, it uses LoadBalancer type, because it needs to be accessible from the outside. All other details are very similar to `mysql.yaml`

1. Deploy the wordpress
    ```
    kubectl create -f wordpress.yaml
    ```

1. Get wordpress service external IP address
    ```
    kubectl get services wordpress
    ```

1. Copy the IP address, and load the page in your browser to view your site.

### Exercise 2 (Optional): Static persistent volume provisioning

1. Delete wordpress persistent volume claim.
1. Manually create a persistent disk in GCE. (Compute engine -> Disks -> Create disk, use `source type = none` to create an empty disk) or use the following command
    ```
    gcloud compute disks create --size=200GB --zone=us-west1-c my-data-disk
    ```
1. Change wordpress deployment to use your persistent disk instead of persistent volume claim. Find `gcePersistentDisk` section in [this](https://kubernetes.io/docs/concepts/storage/volumes/) document for reference.

### Exercise 3 (Optional): Observe how persistent volume is reattached

1. Open wordpress, enter some data.
1. Exec inside mysql pod and kill mysql process.
1. Wait for kubernetes to restart the pod.
1. Make sure that persistend data isn't lost.

### Cleanup

1. Delete all mysql resources
    ```
    kubectl delete -f mysql.yaml
    ```

1. Delete all wordpress resources
    ```
    kubectl delete -f wordpress.yaml
    ```

1. Delete the manually created persistent disk (if created)
    ```
    gcloud compute disks delete --zone=us-west1-c my-data-disk
    ```

Note: We could also cleanup using the wordpress label

---

Next: [Stateful Sets](stateful_sets.md)
