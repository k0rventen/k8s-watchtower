# k8s-watchtower


A minimal k8s observalbility stack providing:
- central application logs aggregation,
- host level & kubernetes metrics,
- RBAC audit log monitoring

_insert dashboards screenshot here_


## deployment

The deployment is made out of 3 steps:
1. Add the audit log function to your cluster
2. Define your audit policy
3. deploy the stack using helm

### Enabling the Audit Log

Depending your your distro the files paths might be different but the gist is:

1. add the following to the APIserver config:

```
--audit-log-maxbackup=3
--audit-log-maxsize=1024
--audit-log-maxage=30
--audit-policy-file=/var/snap/microk8s/current/args/kube-api-audit-policy
--audit-log-path=/var/log/k8s-apiserver-audit.log
```

### Define the audit policy

Create a new file at the path you specified in the `audit-policy-file` argument, which defines your audit policy:
```
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - "RequestReceived"
  - "ResponseStarted"
rules:
- level: Metadata
  namespaces: ["app","app2"]
```

In this file we ask the cluster to log every request at the metadata level, in namespaces `app` and `app2`, once its reply is sent back to the client.

You can refer to https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/ if you want further tweaks.

### deploy the stack

it's recommended to deploy the stack in a separate namespace:

```
helm upgrade --install --namespace watchtower --create-namespace watchtower .
```

Once deployed, port-forward the `grafana` service to your machine
```
kp services/watchtower-grafana -n watchtower 3000:80
```

then access the dashboards at `http://localhost:3000`.


## components