# Install Crossplane on the local cluster

**Create a namespace and enter with kubens**

```shell
kubectl create namespace crossplane-system
kubens crossplane-system
```

**Create project in GCP**

```shell
gcloud projects create <project-id>
```
```shell
gcloud config set project <project-id>
```
```shell
gcloud config get-value project
```

**Create service account for the project**

```shell
gcloud iam service-accounts create <service-account-name> \
--description="Experiments with Crossplane" \
--display-name="crossplane"
```

**Add permissions to the service account**

```shell
gcloud projects add-iam-policy-binding <project-id> \
--member='serviceAccount:<service-account-name>@<project-id>.iam.gserviceaccount.com' \
--role roles/compute.networkAdmin \
--role roles/container.admin \
--role roles/iam.serviceAccountUser
```

**Create JSON key for the service account**

```shell
gcloud iam service-accounts keys create crossplane-key.json \
--iam-account <service-account-name>@<project-id>.iam.gserviceaccount.com
```

**Create and apply secret for authentication using json key**

```shell
cat > authentication.yaml <<EOF
apiVersion: v1
kind: Secret
metadata:
name: gcp-account-creds
namespace: crossplane-system
type: Opaque
data:
credentials: $(base64 -i 'crossplane-key.json' | tr -d "\\n")
EOF
```

```shell
kubectl apply -f authentication.yaml
```

**Install Crossplane**

```shell
curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
```
```shell
sudo mv kubectl-crossplane /opt/homebrew/bin
```
```shell
kubectl crossplane install provider crossplane/provider-gcp:master
```
```shell
kubectl get providers
```

**Configure a GCP cloud ProviderConfig resource in Crossplane** 

```shell
cat > provider-config.yaml <<EOF
apiVersion: gcp.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
name: crossplane-provider-gcp
spec:
projectID: <project-id>
credentials:
source: Secret
secretRef:
namespace: crossplane-system
name: gcp-account-creds
key: credentials
EOF
```

```shell
kubectl apply -f provider-config.yaml
```