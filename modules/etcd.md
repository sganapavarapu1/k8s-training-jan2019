## etcd

etcd is a distributed key-value store that provides a reliable way to store data across a cluster of machines. 

etcd gracefully handles leader elections during network partitions and will tolerate machine failure, including the leader.

It’s open-source and available on [GitHub](https://github.com/coreos/etcd). 

Kubernetes uses etcd to store all persistent information that it needs to operate (cluster state, current and desired state for all pods and deployments, secrets, config maps ...)

### Exercise 1: Manually access etcd 

etcd, as most of the kubernetes system components, runs inside a static pod. This means we can use kubectl to access it.

1. Run the following command to list all system pods.
    ```
    kubectl --namespace kube-system get pods
    ```
    As you might see, there are two etcd pods in the list: `etcd-master` and `etcd-events`. 
    
    The database itself is hosted inside the `etcd-master` pod.

1. Exec inside etcd pod.
    ```
    kubectl --namespace kube-system exec -it etcd-server-master-us-west1-c-ABCD sh
    ```

1. Now you can use `etcdctl` to access the etcd database. The `etcdctl ls` command can be used to navigate inside etcd.
    ```
    etcdctl ls 
    etcdctl ls /registry
    ```
1. List all pods in kube-system namespace.
    ```
    etcdctl ls /registry/pods/kube-system
    ```

1. Select some pod and get its manifest from etcd database.
    ```
    etcdctl get /registry/pods/kube-system/<pod-name>
    ```
    
1. You can use `jq` or an online tool such as [jsonprettyprint](http://jsonprettyprint.com/) to make the manifest more readable.
    ```
    etcdctl get /registry/pods/kube-system/<pod-name>
    ```

    # Piping over to jq
    ```
    kubectl --namespace kube-system exec -it etcd-server-master-us-west1-c-ABCD -- etcdctl get /registry/pods/default/<mypod>|jq 
    ```

### Exercise 2: Backup etcd 

1. Follow the instructions in the etcd documentation to create an etcd backup. [Reference link](https://coreos.com/etcd/docs/latest/v2/admin_guide.html#disaster-recovery) 
1. Deploy etcd on a separate VM [link](https://docs.openstack.org/install-guide/environment-etcd-ubuntu.html), and apply the backup.

