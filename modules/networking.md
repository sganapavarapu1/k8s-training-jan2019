## Networking

### Exercise 1: Installing Kubernetes Addons

1. Create a `simple-service.yaml`

1. Add the following deployment and service to the manifest file.
    ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: simpleservice
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: simpleservice
      template:
        metadata:
          labels:
            app: simpleservice
        spec:
          containers:
          - name: simpleservice
            image: mhausenblas/simpleservice:0.5.0
            ports:
            - containerPort: 9876
            env:
            - name: SIMPLE_SERVICE_VERSION
              value: "0.9"

    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: simpleservice-svc
    spec:
      ports:
        - port: 80
          targetPort: 9876
      selector:
        app: simpleservice
    ```

1. Create the service.
    ```
    kubectl create -f simple-service.yaml
    ```

1. SSH to any of the nodes and examine generated [iptables rules](http://ipset.netfilter.org/iptables.man.html).
    ```
    sudo iptables-save | grep simpleservice
    ```
    The output should resemble this:
    ```
    -A KUBE-SEP-5FXC3Y3RDI3GY23F -s 100.96.2.4/32 -m comment --comment "default/simpleservice-svc:" -j KUBE-MARK-MASQ
    -A KUBE-SEP-5FXC3Y3RDI3GY23F -p tcp -m comment --comment "default/simpleservice-svc:" -m tcp -j DNAT --to-destination 100.96.2.4:9876
    -A KUBE-SEP-UL7W7MQRDB5MWDF3 -s 100.96.2.3/32 -m comment --comment "default/simpleservice-svc:" -j KUBE-MARK-MASQ
    -A KUBE-SEP-UL7W7MQRDB5MWDF3 -p tcp -m comment --comment "default/simpleservice-svc:" -m tcp -j DNAT --to-destination 100.96.2.3:9876

    -A KUBE-SERVICES ! -s 100.96.0.0/11 -d 100.67.234.205/32 -p tcp -m comment --comment "default/simpleservice-svc: cluster IP" -m tcp --dport 80 -j KUBE-MARK-MASQ
    -A KUBE-SERVICES -d 100.67.234.205/32 -p tcp -m comment --comment "default/simpleservice-svc: cluster IP" -m tcp --dport 80 -j KUBE-SVC-6GA5KCFGZYCRFQCY
    -A KUBE-SVC-6GA5KCFGZYCRFQCY -m comment --comment "default/simpleservice-svc:" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-UL7W7MQRDB5MWDF3
    -A KUBE-SVC-6GA5KCFGZYCRFQCY -m comment --comment "default/simpleservice-svc:" -j KUBE-SEP-5FXC3Y3RDI3GY23F
    ```
    See networking slides for more detail.

### Exercise 2: Track iptables changes while redeploying the service

Redeploy the service in different configurations and observe change to iptables. Make sure you understand the changes. Use `sudo iptables-save | grep simpleservice` command to keep track of the relevant iptables rules.

Try the following configurations.

1. Scale down the number of pods, covered by the service, to 1.
1. Scale up the number of pods, covered by the service, to 3.
1. Change service type to NodePort.
1. Change service type to LoadBalancer.

## Cleanup.

```
kubectl delete -f simple-service.yaml
```
