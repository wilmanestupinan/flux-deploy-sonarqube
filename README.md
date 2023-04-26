
# Install the Flux CLI
brew install fluxcd/tap/flux

# Export your credentials
export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-username>


# Validate instalation flux
flux check --pre

flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=flux-deploy-github \
  --branch=main \
  --path=./clusters/sonarqube \
  --personal

flux bootstrap github --owner=$GITHUB_USER --repository=flux-deploy-github --path=./clusters/sonarqube  --reconcile 

 flux uninstall --keep-namespace 

flux bootstrap github \
  --components-extra=image-reflector-controller,image-automation-controller \
  --owner=$GITHUB_USER \
  --repository=flux-deploy-sonarqube \
  --branch=main \
  --path=clusters/sonarqube \
  --read-write-key \
  --personal


flux create source git podinfo \
--url=https://github.com/stefanprodan/podinfo \
--branch=main \
--interval=30s \
--export > ./clusters/sonarqube/podinfo-source.yaml

flux create kustomization podinfo \
  --target-namespace=default \
  --source=podinfo \
  --path="./kustomize" \
  --prune=true \
  --interval=5m \
  --export > ./clusters/sonarqube/podinfo-kustomization.yaml

  (

## Error to delete namespace in kubernetes when it is terminating state

NAMESPACE=flux-system \
kubectl proxy & \
kubectl get namespace $NAMESPACE -o json |jq '.spec = {"finalizers":[]}' >temp.json \
curl -k -H "Content-Type: application/json" -X PUT --data-binary @temp.json 127.0.0.1:8001/api/v1/namespaces/$NAMESPACE/finalize)


aws sts get-caller-identity --profile ic-dev

aws eks list-clusters --profile ic-dev

aws eks --region us-east-1 update-kubeconfig --name eks-dev-1 > dev.yaml --profile ic-dev

export KUBECONFIG=$KUBECONFIG:~/.kube/config:shared.yaml:dev.yaml



  # Print the reconciliation logs of all Flux custom resources in your cluster
  flux logs --all-namespaces
  
  # Print all logs of all Flux custom resources newer than 2 minutes
  flux logs --all-namespaces --since=2m

  # Stream logs for a particular log level
  flux logs --follow --level=error --all-namespaces

  # Filter logs by kind, name and namespace
  flux logs --kind=Kustomization --name=podinfo --namespace=default

  # Print logs when Flux is installed in a different namespace than flux-system
  flux logs --flux-namespace=my-namespace