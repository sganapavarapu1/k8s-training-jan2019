# Security

## Objectives

- Implement PSP to limit container capabilities
- Implement network policy to limit interactions between pods
- Implement image security scanning to prevent using outdated versions of the base image

## Pre-requisites

- [Enable pod security policies (PSP)](enable_psp_on_kops.md)
- Use CNI weave/calico networking instead of kubenet

## Exercise 01: Pod Security Policy

Create a PSP file called `psp.yaml`:

```
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: 'docker/default'
    apparmor.security.beta.kubernetes.io/allowedProfileNames: 'runtime/default'
    seccomp.security.alpha.kubernetes.io/defaultProfileName:  'docker/default'
    apparmor.security.beta.kubernetes.io/defaultProfileName:  'runtime/default'
spec:
  privileged: false
  # Required to prevent escalations to root.
  allowPrivilegeEscalation: false
  # This is redundant with non-root + disallow privilege escalation,
  # but we can provide it for defense in depth.
  requiredDropCapabilities:
    - ALL
  # Allow core volume types.
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    # Assume that persistentVolumes set up by the cluster admin are safe to use.
    - 'persistentVolumeClaim'
  hostNetwork: false
  hostIPC: false
  hostPID: false
  runAsUser:
    # Require the container to run without root privileges.
    rule: 'MustRunAsNonRoot'
  seLinux:
    # This policy assumes the nodes are using AppArmor rather than SELinux.
    rule: 'RunAsAny'
  supplementalGroups:
    rule: 'MustRunAs'
    ranges:
      # Forbid adding the root group.
      - min: 1
        max: 65535
  fsGroup:
    rule: 'MustRunAs'
    ranges:
      # Forbid adding the root group.
      - min: 1
        max: 65535
  readOnlyRootFilesystem: false
```

```
kubectl apply -f psp.yaml
```

Typically `Pods` are created by `Deployments`, `ReplicaSets`, not by the user directly. We need to grant permissions for using this policy to the default account.

Create a role called `role.yaml`:
```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: pod-starter
rules:
- apiGroups:
  - extensions
  resources:
  - podsecuritypolicies
  resourceNames:
  - restricted
  verbs:
  - use
```

```
kubectl apply -f role.yaml
```

Create rolebinding `binding.yaml`:
```
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: pod-starter-binding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: pod-starter
subjects:
# A specific service account in my-namespace
- kind: ServiceAccount # Omit apiGroup
  name: default
  namespace: default
```

```
kubectl apply -f binding.yaml
```

Now try to create a priviledged container:

```
kubectl create -f- <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: privileged
  labels:
    app: privileged
spec:
  replicas: 1
  selector:
    matchLabels:
      app: privileged
  template:
    metadata:
      labels:
        app: privileged
    spec:
      containers:
        - name:  pause
          image: k8s.gcr.io/pause
          securityContext:
            privileged: true
EOF
```

`Deployment` creates `ReplicaSet` that in turn creates `Pod`. Let' see the `ReplicaSet` state.

```
kubectl get rs -l=app=privileged
```

```
NAME                    DESIRED   CURRENT   READY     AGE
privileged-6c96db7488   1         0         0         5m
```

No pods created. Why?

```
kubectl describe rs -l=app=privileged
```

```
..
Error creating: pods "privileged-6c96db7488-" is forbidden: unable to validate against any pod security policy: [spec.containers[0].securityContext.privileged: Invalid value: true: Privileged containers are not allowed]
```

Admission controller forbids creating priviledged container as the applied policy states.

What happens if you create pod directly?

```
kubectl create -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: privileged
spec:
  containers:
    - name:  pause
      image: k8s.gcr.io/pause
      securityContext:
        privileged: true
EOF
```

Try it and explain the result.

## Exercise 02: Network Policy

Let's see how to use network policy for blocking the external traffic for a `Pod`

Create file called `deny-egress.yaml`:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: foo-deny-egress
spec:
  podSelector:
    matchLabels:
      app: foo
  policyTypes:
  - Egress
  egress:
  # allow DNS resolution
  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
```

```
kubectl apply -f deny-egress.yaml
```

This file blocks all the outgoing traffic except DNS resolution.

Now start the pod that matches label `app=foo`

```
kubectl run --rm --restart=Never --image=alpine -i -t -l app=foo test -- ash
```

In container run:
```
wget --timeout 1 -O- http://www.example.com
```

```
Connecting to www.example.com (93.184.216.34:80)
wget: download timed out
```

You see the name resolution works fine but external connections are dropped.

### Cleanup

1. Delete the network policy and role binding
    ```
    kubectl delete -f deny-egress.yaml
    kubectl delete -f binding.yaml
    ```
1. Disable PSPs on the cluster
    ```
    kops edit cluster # Remove kubeAPIServer
    kops update cluster --yes
    kops rolling-update cluster --yes
    ```
