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

# 3. Access The Argo CD API Server

By default, the Argo CD API server is not exposed with an external IP. To access the API server, choose one of the following techniques to expose the Argo CD API server:

## Service Type Load Balancer
Change the argocd-server service type to LoadBalancer:
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

## Service Type NodePort
Change the argocd-server service type to NodePort:
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'

kubectl get svc -n argocd
NAME                                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP   10.101.215.204   <none>        7000/TCP,8080/TCP            21h
argocd-dex-server                         ClusterIP   10.98.152.125    <none>        5556/TCP,5557/TCP,5558/TCP   21h
argocd-metrics                            ClusterIP   10.97.54.217     <none>        8082/TCP                     21h
argocd-notifications-controller-metrics   ClusterIP   10.101.150.72    <none>        9001/TCP                     21h
argocd-redis                              ClusterIP   10.108.99.28     <none>        6379/TCP                     21h
argocd-repo-server                        ClusterIP   10.102.142.52    <none>        8081/TCP,8084/TCP            21h
argocd-server                             NodePort    10.98.206.27     <none>        80:32705/TCP,443:31612/TCP   21h
argocd-server-metrics                     ClusterIP   10.107.145.17    <none>        8083/TCP                     21h
```

## Port Forwarding¶
Kubectl port-forwarding can also be used to connect to the API server without exposing the service.
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
The API server can then be accessed using
```bash
https://localhost:8080
```

# 4 Get the Argocd Password
You get the password by typing
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
## Login Using The CLI

The initial password for the admin account is auto-generated and stored as clear text in the field password in a secret named argocd-initial-admin-secret in your Argo CD installation namespace. You can simply retrieve this password using the argocd CLI: 
argocd admin initial-password -n argocd
```bash
argocd admin initial-password -n argocd
uyqteKM4VH2l9kGw

 This password must be only used for first time login. We strongly recommend you update the password using `argocd account update-password`.
```
```bash
You should delete the argocd-initial-admin-secret from the Argo CD namespace once you changed the password. The secret serves no other purpose than to store the initially generated password in clear and can safely be deleted at any time. It will be re-created on demand by Argo CD if a new admin password must be re-generated.

Here’s a breakdown of the key points:

- Initial Purpose: The argocd-initial-admin-secret is created when Argo CD is first set up. It holds the default password for the admin account in clear text.

- Changing the Password: After you change the admin password (usually through the Argo CD CLI or UI), the secret with the initial password becomes redundant. It contains sensitive information that should no longer be used.

- Safety and Cleanup: It's recommended to delete this secret after changing the password to improve security. The initial password stored in the secret is no longer needed, and keeping it could pose a security risk if it's inadvertently exposed.

- Re-creation: If Argo CD needs to generate a new admin password in the future, it will create a new argocd-initial-admin-secret automatically. So, deleting the old one won't affect your ability to manage Argo CD or change passwords later.

In summary, once you have updated the admin password, deleting the argocd-initial-admin-secret is a good practice to remove outdated and potentially insecure information from your Kubernetes cluster.
```
