## Kubelet

The kubelet is the primary “node agent” that runs on each node. 

The kubelet takes a set of PodSpecs that are provided through various mechanisms (primarily through the apiserver) and ensures  the containers described in those PodSpecs are running and healthy. 

### Exercise 1: Investigate kubelet 

1. SSH to the master node.
    * In the GCP Console, go to the VM Instances page.
    * In the list of virtual machine instances, click SSH in the row of the master instance.

1. Check the kubelet service status
    ```
    systemctl status kubelet
    ```
    The status should be `active(running)`.

1. Check the kubelet startup parameters 

    Save output of the previous commend to a file.
    ```
    systemctl status kubelet > /tmp/kubelet-params
    ``` 
    Open `/tmp/kubelet-params` and check statup parameters. A few important parameters are copied here.
    
    * `--cluster-dns=100.64.0.10` - DNS server used for all pods, point to Kube DNS system pod.
    * `--kubeconfig=/var/lib/kubelet/kubeconfig` - kubeconfig is used to connecto to kube API
    * `--network-plugin=kubenet` - use [kubenet](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#kubenet) network plugin
    * `--pod-manifest-path=/etc/kubernetes/manifests` - the folder where all static pods are located.

1. Check the kubelet logs
    ```
    sudo journalctl -u kubelet
    ```
    You don't need to understand everything from here, just remember how to access these logs for troubleshooting.

### Exercise 2 (Optional): Run a static pod 

1. Put your own pod manifest into `--pod-manifest-path` folder. (The folder should be watched every 20s, so no need to restart kubelet)
1. Check whether kubernetes will run your pod. 

