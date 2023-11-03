# openshift-gitops-apps-any-ns
Apps in any Namespace with OpenShift GitOps Operator.

Current status: Not working, this is just a WIP.

- ArgoCD console does not show the app when is created inside the myapps-acme
  Namespace.
- Application not synced.
```bash
oc get application -n myapps-acme
NAME          SYNC STATUS   HEALTH STATUS
test-any-ns
```

# Known errors

## OpenShift users can see clusters or projects

When logged in the ArgoCD instance using the OpenShift logging,
with a cluster-admin user, I cant see anything in the UI, nor clusters
nor projects.

I have to login usen the local `admin` user of ArgoCD.

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