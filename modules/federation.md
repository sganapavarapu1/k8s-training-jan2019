## Federations

## Exercise 1: Create federated cluster 

1. Install kubefed
    * Download kubefed from this page https://github.com/kubernetes/federation/releases (Download doesn't work from the command line, because you need to be authenticated with gcp account.)
    * Extract the archive and copy `kubefed` executable file to the Cloud Shell VM. Or, alternatively, copy `~/.kube/config` file to your local machine and work from there.

1. Deploy a new cluster using kops. Use `Day 1 -> Installing a Kubernetes Cluster on Google Cloud Platform (GCP) with Kubernetes Operations (kops)`   lesson for reference. Cluster name should be `new.k8s.local` 

1.  Select your host cluster. 
    * Run `kubectl config get-contexts` to list all available contexts and check what context is curently used. (You should see at least 2 contexts, corresponding to both your clusters)
    * Use `kubectl config use-context simple.k8s.local` command to set current context to `simple.k8s.local`.

1. Create GCP managed DNS zone
    ```
    gcloud dns managed-zones create federation   --description "Kubernetes federation testing"   --dns-name <some-unique-dns-name> 
    ```

1. Deploy a federation control plane  
    ```
    kubefed init fellowship \
        --host-cluster-context=simple.k8s.local \
        --dns-provider="google-clouddns" \
        --dns-zone-name="<same-dns-name-as-you-use-in-the-previous-command>."
    ```
    Don't remove '.' at the end of the dns name.

1. Use `kubectl config use-context fellowship` to switch to the federation context.

1. Join the second cluster.
    ```
    kubefed join new  --cluster-context=new.k8s.local  --host-cluster-context=simple.k8s.local
    ```

1. List clusters
    ```
    kubectl get clusters
    ```
    You should see that `new` cluster is joined.

## Exercise 2: Deploy a multi cloud application

1. Make sure you are still using `fellowship` context.

1. Make a deployment. (We are deploying to a federation API which is different from kubernetes API, use [this](https://github.com/madeden/blogposts/blob/master/k8s-federation/src/manifests/microbots-ds.yaml) deployment as a sample)

