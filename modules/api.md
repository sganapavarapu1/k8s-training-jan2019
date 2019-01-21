## API Server

The API Server services REST operations and provides the frontend to the cluster’s shared state through which all other components interact.

### Exercise 1: Manually access kubernetes API

1. Run kubectl as a proxy on a Cloud Shell VM.
    ```
    kubectl proxy --port=8080
    ``` 
    This should expose the API on `localhost:8080`. This method of accessing the API has an advantage that you don't need to authenticate when accessing API.

1. Forward port 8080 from the Cloud Shell to your local machine. 

    From Cloud Shell top bar select option `Preview on port 8080` 
    
    This should open in your browser a page with all top-level API urls. Now you can access API from Cloud Shell, using localhost:8080, or from you local machine using the host that you can copy from the previously opened page.

1. From the Cloud Shell, execute the following command to query all pods in default namespace
    ```
    curl localhost:8080/api/v1/namespaces/default/pods
    ```

### Exercise 2 (Optional): Deploy a pod using API 

1. Check the [API documentation](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.10/#create-55) to see the structure of the json object that needs to be posted to the API in order to deploy a pod. (The structure is exactly the same as the structure of the yaml pod definition, you can use any yaml to json converter to get required object)
1. Post your json to the API server in order to deploy a pod. (Use `curl -X POST -d '<your json file>'` to execute a POST request) 
1. Check that your pod is deployed successfully.

