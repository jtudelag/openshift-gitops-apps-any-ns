# openshift-gitops-apps-any-ns
Apps in any Namespace with OpenShift GitOps Operator.

This is tech preview sinc 1.7, but is not documented...only briefly mentioned
in the [release note](https://docs.openshift.com/container-platform/4.10/cicd/gitops/gitops-release-notes.html#gitops-release-notes-1-7-0_gitops-release-notes).

``````
With this update, you can create applications, which are managed by the same control plane Argo CD instance, in any namespace in the same cluster. As an administrator, perform the following actions to enable this update:

* Add the namespace to the .spec.sourceNamespaces attribute for a cluster-scoped Argo CD instance that manages the application.

* Add the namespace to the .spec.sourceNamespaces attribute in the AppProject custom resource that is associated with the application.
``````

## Pre-reqs per upstream docs

* ArgoCD must be [cluster scoped](https://argo-cd.readthedocs.io/en/stable/operator-manual/app-any-namespace/#cluster-scoped-argo-cd-installation). For OpenShift GitOps that
means  setting the env var `ARGOCD_CLUSTER_CONFIG_NAMESPACES` as described in this [article](https://developers.redhat.com/articles/2023/03/06/5-global-environment-variables-provided-openshift-gitops#5_environment_variables__details).

* Swith ArgoCD to [resource tracking](https://argo-cd.readthedocs.io/en/stable/operator-manual/app-any-namespace/#switch-resource-tracking-method). This is not documented
for OpenShift GitOps neither...It has to be set in the ArgoCD CR `spec.resourceTrackingMethod: annotation`,

# Known Issues

## OpenShift users can NOT see clusters or projects

When logged in the ArgoCD instance using the OpenShift logging,
with a cluster-admin user, I cant see anything in the UI, nor clusters
nor projects.

Due to one or both reasons:

  * Default ArgoCD instace permissions [narrowing since version 1.10](https://issues.redhat.com/browse/GITOPS-3032).
  * My user does not belonging to the [cluster-admin group](https://docs.openshift.com/gitops/1.10/accesscontrol_usermanagement/configuring-sso-on-argo-cd-using-dex.html#gitops-dex-role-mappings_configuring-sso-for-argo-cd-using-dex).

## Not enough permissions to deploy inside hello-openshift
Apparently the ArgoCD instance inside openshift-gitops does not have
enough permissions inside the hello-openshift ns:

```
Failed sync attempt to e079dda605cdc6b1c44e5a6343b5aad76ce8414e: one or more objects failed to apply, reason: deployments.apps is forbidden: User "system:serviceaccount:app-gitops:argo-app-any-ns-argocd-application-controller" cannot create resource "deployments" in API group "apps" in the namespace "hello-openshift"
```

You can fix it using oc:
```bash
oc -n hello-openshift adm policy add-role-to-user cluster-admin system:serviceaccount:app-gitops:argo-app-any-ns-argocd-application-controller
```

Or applying the role binding `4_hello-openshift-rb.yaml` inside the
`3_argocd_app` folder.

I would expect a cluster scoped ArgoCD to be able to create a Deployment on a Namespace,
but I am probably missing sth and there is a good explanation for it.

## OpenShift GitOps Operator ignores wildcards in sourceNamespaces

Altough [ArgoCD upstream supports it](https://argo-cd.readthedocs.io/en/stable/operator-manual/app-any-namespace/#change-workload-startup-parameters) there is an issue for
[OpenShift GitOps open](https://issues.redhat.com/browse/RFE-4535).