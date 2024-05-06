# argocd-example-application

## Installation
- Openshift Pipelines
- Openshift GitOps

```
oc apply -f ./cluster-configuration
```
Wait till both operators are installed

Configure Argo CD instance with an account that can access the API

```
oc apply -f ./argocd-configuration
```

## Argo CD diff with a pipeline

### Get Argo CD configuration
Get Argo CD server address.
```
oc -n openshift-gitops get route openshift-gitops-server -o jsonpath='{.spec.host}'
```

Get Argo CD admin password:
```
oc extract secret/openshift-gitops-cluster -n openshift-gitops --to=-
```

Then generate a new token for the account pipeline-account
```
argocd login <<ArgCD-server> --username admin --password <<ArgCD-password>>
argocd account generate-token --account pipeline-account
```

Set the new token in the value argocd.toke
This value is set in the pullrequest/pipeline/argocd-env-secret.yaml

### Deploy pipelines

```
helm upgrade pipelines ./pullrequest/pipeline/ --install --set argocd.token=<<ArgCD-token>>
```

### Deploy application Set

```
oc apply -f argocd/appset-discount-kustomize.yaml
oc apply -f argocd/appset-discount-helm.yaml
```

Synchronize all the Argo CD applications

### Create a new branch with changes

Create a new branch called diff.
Make changes in the applications configurations, folders "discounts-helm" and "discouts-kustomize".

### Start pipeline

Execute the pull request pipeline with the branch diff. We will see the Argo CD diff

```
tkn pipeline start pull-request-pipeline -n ci --param revision=diff --param source-repo=https://github.com/davidseve/argocd-example-application  --workspace name=diff-result,claimName=workspace-pvc-diff-result --workspace name=source-folder,claimName=workspace-pvc-source-folder
```
## Argo CD diff manual

- Create an application in Argo CD
- Create a new branch called diff
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
