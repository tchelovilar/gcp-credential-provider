# gcp-credential-provider

Credential provider plugin for Kubernetes Kubelet. Provide Google Artifact Registry and GCR credentials for [Kubelet image credential provider](https://kubernetes.io/docs/tasks/administer-cluster/kubelet-credential-provider/), it can be used in Bare Metal Kubernetes cluster to automatically pull container images from private repositories.

## Usage

Every node in the cluster needs that steps in order to pull the container images.

** Pre-requirements**
- Python
- Google Credentials with Artifact or GCR access

1. Install the [docker-credential-gcr](https://github.com/GoogleCloudPlatform/docker-credential-gcr) helper

This helper is used to generate container registry credentials without gcloud framework

```bash
VERSION=2.1.8
OS=linux
ARCH=amd64
curl -fsSL "https://github.com/GoogleCloudPlatform/docker-credential-gcr/releases/download/v${VERSION}/docker-credential-gcr_${OS}_${ARCH}-${VERSION}.tar.gz" \
        | tar xz docker-credential-gcr \
    && chmod +x docker-credential-gcr \
    && sudo mv docker-credential-gcr /usr/bin/
```

2. Install the `gcp-credential-provider`

```bash
CREDENTIAL_PROVIDER_PATH=/opt/credential_provider
git clone https://github.com/tchelovilar/gcp-credential-provider.git
pip install -r gcp-credential-provider/requirements.txt
mkdir -p $CREDENTIAL_PROVIDER_PATH
cp gcp-credential-provider/src/gcp-credential-provider $CREDENTIAL_PROVIDER_PATH
chmod +x "$CREDENTIAL_PROVIDER_PATH/gcp-credential-provider" 
```

3. Add the extra arguments to the Kubelet

```
    --image-credential-provider-config=/etc/kubernetes/credential-provider.yaml
    --image-credential-provider-bin-dir=/opt/credential_provider
```

** For `kubespray` you can define the argument `kubelet_custom_flags`

```
kubelet_custom_flags:
  - "--image-credential-provider-config=/etc/kubernetes/credential-provider.yaml"
  - "--image-credential-provider-bin-dir=/opt/credential_provider"
```

4. Create the `/etc/kubernetes/credential-provider.yaml` file with the content

```yaml
# Works with kubernetes from 1.26
apiVersion: kubelet.config.k8s.io/v1
kind: CredentialProviderConfig
providers:
- name: gcp-credential-provider
  matchImages:
    - "*.pkg.dev"
    - "eu.gcr.io"
    - "gcr.io"
  defaultCacheDuration: "10m"
  apiVersion: credentialprovider.kubelet.k8s.io/v1
#   # For debuging purposes
#   args:
#   - -v
#   - --log-file=/tmp/cred.log
  env:
    - name: GOOGLE_APPLICATION_CREDENTIALS
      value: /credential/path
```


# Reference:
- https://kubernetes.io/docs/tasks/administer-cluster/kubelet-credential-provider/
- https://kubernetes.io/blog/2022/12/22/kubelet-credential-providers/
- https://kubernetes.io/docs/reference/config-api/kubelet-credentialprovider.v1/