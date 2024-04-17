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

argocd login --name admin --password EWMt4mnpfhF9DY1V6zcPrsuweHqKoLx7 $ARGOCD_ROUTE

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

Also update he file pullrequest/pipeline/argocd-env-secret.yaml with Argo CD user and password or token.

Use this command to get admin password:
```
oc extract secret/openshift-gitops-cluster -n openshift-gitops --to=-
```

### Deploy pipelines

```
oc apply -f ./pullrequest/pipeline/
```