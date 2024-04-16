# argocd-example-application

## Installation
- Openshift GitOps

## Argo CD diff demo

- Create an application in Argo CD
- Create a new branch called diff-branch
- Change the number of replicas in staging deployment.yaml
- Commit and push changed to diff-branch

´´´
ARGOCD_ROUTE=$(oc -n openshift-gitops get route openshift-gitops-server -o jsonpath='{.spec.host}')

argocd login --name admin --password EWMt4mnpfhF9DY1V6zcPrsuweHqKoLx7 $ARGOCD_ROUTE

argocd app diff openshift-gitops/staging-discounts --revision  rama-delete
´´´

Out put should be like this:

´´´
===== apps/Deployment openshift-gitops/discounts ======
134c134
<   replicas: 1
---
>   replicas: 8

´´´

