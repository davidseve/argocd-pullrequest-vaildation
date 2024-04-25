# argocd-example-application

## Installation
- Openshift Pipelines
- Openshift GitOps

```
oc apply -f ./cluster-configuration
```
Wait till both operators are installed

## Argo CD diff demo manual

- Create an application in Argo CD
- Create a new branch called diff-branch
- Change the number of replicas in staging deployment.yaml
- Commit and push changed to diff-branch

```
ARGOCD_ROUTE=$(oc -n openshift-gitops get route openshift-gitops-server -o jsonpath='{.spec.host}')

argocd login --name admin --password <<CHANGE_ME>> $ARGOCD_ROUTE

argocd app diff openshift-gitops/staging-discounts --revision  diff-branch
```

Out put should be like this:

```
===== apps/Deployment openshift-gitops/discounts ======
134c134
<   replicas: 1
---
>   replicas: 8
```

## Argo CD diff demo pipeline

### Get Argo CD configuration
Get Argo CD server address and update the file pullrequest/pipeline/argocd-env-configmap.yaml

```
oc -n openshift-gitops get route openshift-gitops-server -o jsonpath='{.spec.host}'
```

Also update the file pullrequest/pipeline/argocd-env-secret.yaml with Argo CD user and password or token.

Use this command to get admin password:
```
oc extract secret/openshift-gitops-cluster -n openshift-gitops --to=-
```

### Deploy application Set

```
oc apply -f argocd/appset-discount-kustomize.yaml
oc apply -f argocd/appset-discount-helm.yaml
```

### Deploy pipelines

```
helm upgrade pipelines ./pullrequest/pipeline/ --set argocd.pass=<<ArgCD-password>> --install
```

### Start pipeline

```
tkn pipeline start pull-request-pipeline -n ci --param revision=diff --param source-repo=https://github.com/davidseve/argocd-example-application  --workspace name=diff-result,claimName=workspace-pvc-diff-result --workspace name=source-folder,claimName=workspace-pvc-source-folder
```

