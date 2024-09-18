# 1. Install Argo CD
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
# 2. Download Argo CD CLI

First, retrieve the version of the current release (or set the ARGOCD_VERSION environment variable manually):  
```bash
ARGOCD_VERSION=$(curl --silent "https://api.github.com/repos/argoproj/argo-cd/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')
```
Then, retrieve the binary from GitHub to a temporary location:
```bash
curl -sSL -o /tmp/argocd-${ARGOCD_VERSION} https://github.com/argoproj/argo-cd/releases/download/${ARGOCD_VERSION}/argocd-linux-amd64
```
Finally, make the binary executable and move it to a location within your $PATH, in this example /usr/local/bin:
```bash
chmod +x /tmp/argocd-v2.12.3
sudo mv /tmp/argocd-v2.12.3 /usr/local/bin/argocd 
```
Verify that your CLI is working properly:
```bash
argocd version --client
```
This should give an output similar to the following (details may differ across versions and platform):
```bash
argocd: v1.8.1+c2547dc
  BuildDate: 2020-12-10T02:57:57Z
  GitCommit: c2547dca95437fdbb4d1e984b0592e6b9110d37f
  GitTreeState: clean
  GoVersion: go1.14.12
  Compiler: gc
  Platform: linux/amd64
```

