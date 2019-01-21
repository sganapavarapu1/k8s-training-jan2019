## Manage infrastructure using terraform 

### Exercise 1: Manually deploy new cluster using terraform 

1. Run the following command to create a new kubernetes cluster and export its terraform state.

    ```
    mkdir terraform
    cd terraform
    export KOPS_FEATURE_FLAGS=AlphaAllowGCE
    kops create cluster \
      --name=new.k8s.local \
      --zones us-east1-c \
      --project=${DEVSHELL_PROJECT_ID} \
      --out=. \
      --target=terraform
    ```

1. Change the default network name. This step is required because we are deploying the second cluster inside the same project.

    In the `kubernetes.tf` find the following element
    ```
    resource "google_compute_network" "default" {
      name                    = "default"
      auto_create_subnetworks = true
    }
    ```
    Change network name to `new-default`. The element should look like this:
    ```
    resource "google_compute_network" "default" {
      name                    = "new-default"
      auto_create_subnetworks = true
    }
    ```

1. Install terraform.
    ```
    wget https://releases.hashicorp.com/terraform/0.11.7/terraform_0.11.7_linux_amd64.zip
    unzip terraform_*
    mv terraform ~/bin/
    rm terraform_*.zip
    ```

1. Execute the following command to see what GCE infrastructure is going to be created.
    ```
    terraform init
    terraform plan
    ```

1. Deploy the new cluster.
    ```
    terraform apply
    ```

1. Export the cluster credentials
    ```
    kops export kubecfg new.k8s.local
    ```

1. Check whether the cluster is healthy 
    ```
    kubectl cluster-info
    kubectl get nodes
    ```

1. Oops, at this point you will probably see a connection error. The error is expected and is caused by a bug in kops terraform integration. Now we will try to troubleshoot the cluster.

### Exercise 2 (Optional): Advanced cluster troubleshooting

At this point, we don't want to explain the cause of the issue, but instead want you to troubleshoot it on your own. 

A few clues for you:

1. Check Network services -> Load balancing page to make sure the API load balancer is set up correctly and redirects traffic to the kubernetes master node.  Either from this page or from terraform file try to figure out to what port at the master node load balancer redirects traffic.
1. If the load balancer is set up correctly, SSH to the master node and use `netstat -tulpn` to see whether something is listening on the port you've just discovered.  If not, this means something is probably wrong with the kubernetes API server.
1. All kubernetes master components except kubelet run inside docker.  Use the `docker ps` command to list them.  What do you see?  What components are missing or unhealthy?
1. If the problem is related to kubelet, you may want to look at kubelet logs.  Kubelet runs as a `systemd` service.  Use `sudo journalctl -u kubelet` to list all kubelet logs.
1. Kubelet is responsible for starting all other components.  They are defined as pods.  If some of the components are missing or misconfigured you may want to take a look at the `/etc/kubernetes/manifests` folder to see the definition of all components.  Is there anything suspicious there?
1. If some of the components are unhealthy you can take a look at their logs. They can be found in the `/var/log` folder.
1. There are two important systemd services that are not part of kubernetes, but are deployed by kops.  These services are called `kops-configuration` and `protokube`.  They are responsible for configuring kubelet, starting it, and updating its configuration when you update or upgrade the cluster.  Use `journalctl` to examine their logs.
1. Good luck!  If you are not be able to troubleshoot the issue, we'll show you the solution a little bit later.
