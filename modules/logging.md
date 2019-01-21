## Logging

Setting up an EFK stack on Kubernetes.

### Exercise 1: Installing the Kubernetes elasticsearch logging add-on

1. Install the [Elasticsearch Logging](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch) add-on from the [kops repo](https://github.com/kubernetes/kops/tree/master/addons/logging-elasticsearch)
    ```
    kubectl create -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/logging-elasticsearch/v1.6.0.yaml
    ```

    Review the yaml file carefully and verify all objects have been created successfully.  This will add Fluentd for parsing/shipping, Elasticsearch for storing/indexing and Kibana for visualization.

    >Note: There will be an error creating one object.  You will need to troubleshoot the failure, update the yaml locally and redeploy.

1. Start a proxy on port 8080.
    ```
    kubectl proxy -p 8080
    ```
    
1. Forward port 8080 from the Cloud Shell to your local machine.

    From Cloud Shell top menu bar select option `Preview on port 8080`

1. View the Kibana service on the /proxy/ endpoint.

    Append `api/v1/namespaces/kube-system/services/kibana-logging/proxy` to the top level domain.
    
    E.g. `https://8080-dot-3438793-dot-devshell.appspot.com/api/v1/namespaces/kube-system/services/kibana-logging/proxy`

1. Explore the Kibana Dashboard and what information you can find there.

### Exercise 2 (Optional): Setup a Kibana Dashboard for Kube-System

1. Create a dashboard to quickly identify the type of error we saw in the previous exercise.