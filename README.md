
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

## Error to delete namespace in kuberneten when it is terminating state
NAMESPACE=flux-system \
kubectl proxy & \
kubectl get namespace $NAMESPACE -o json |jq '.spec = {"finalizers":[]}' >temp.json \
curl -k -H "Content-Type: application/json" -X PUT --data-binary @temp.json 127.0.0.1:8001/api/v1/namespaces/$NAMESPACE/finalize)