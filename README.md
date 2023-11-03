# openshift-gitops-apps-any-ns
Apps in any Namespace with OpenShift GitOps Operator


# Known errors

## Not enough permissions to deploy inside hello-openshift
Apparently the ArgoCD instance inside openshift-gitops does not have
enough permissions inside the hello-openshift ns:

```
Failed sync attempt to e079dda605cdc6b1c44e5a6343b5aad76ce8414e: one or more objects failed to apply, reason: deployments.apps is forbidden: User "system:serviceaccount:openshift-gitops:argo-app-any-ns-argocd-application-controller" cannot create resource "deployments" in API group "apps" in the namespace "hello-openshift"
```

You can fix it using oc:
```bash
oc -n hello-openshift adm policy add-role-to-user cluster-admin system:serviceaccount:openshift-gitops:argo-app-any-ns-argocd-application-controller
```

Or applying the role binding `4_hello-openshift-rb.yaml` inside the 
`3_argocd_app` folder.