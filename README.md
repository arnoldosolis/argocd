# Argo CD Setup with Helm
This guide installs Argo CD using Helm, with configuration defined in a values file so you’re not relying on CLI flags or manual patches.
---
## 📦 Prerequisites
* Kubernetes cluster (k3s is fine)
* `kubectl` configured
* `helm` installed
Verify:
```bash
kubectl get nodes
helm version
```
---
## 1️⃣ Add the Argo Helm Repository
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```
---

## 2️⃣ Create a Values File (IMPORTANT)
Instead of using `--set`, define everything declaratively.
Create `values.yaml`:
```yaml
server:
  service:
    type: NodePort

configs:
  params:
    server.insecure: true
```
You can expand this later (ingress, RBAC, etc.)
---

## 3️⃣ Install Argo CD via Helm
```bash
helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace \
  -f values.yaml
```
This:
* Creates the namespace ✅
* Applies your config ✅
---

## 4️⃣ Verify Installation
```bash
kubectl get pods -n argocd
```
Wait until everything is `Running`.
---

## 5️⃣ Access Argo CD
### Get NodePort
```bash
kubectl get svc argocd-server -n argocd
```
Then:
```
https://<node-ip>:<nodeport>
```
---

## 6️⃣ Get Admin Password
```bash
kubectl get secret argocd-initial-admin-secret \
  -n argocd \
  -o jsonpath="{.data.password}" | base64 -d && echo
```
* Username: `admin`
---

## 🧠 Key Takeaways
* Helm is used **once to install Argo CD**
* Configuration lives in `values.yaml`, not CLI flags
* After install, you can move to full GitOps
---

## ⚠️ Gotchas
* Don’t mix `--set` and `values.yaml` long-term
* Avoid manual `kubectl patch`
* Be cautious enabling self-management without matching config
---

## 🚀 Next Steps

* Add Ingress (NGINX instead of NodePort)
* Add your first app
* Introduce App of Apps pattern
 
*If for some reason you need reach ui via port forward, feel free to do the following:*
```
kubectl port-forward svc/argocd-server 8080:443 -n argocd --address 0.0.0.0
```

*When you find yourself needing to update something related to argocd, go to values.yaml and the run the following command to update(upgrade)*
```
helm upgrade argocd argo/argo-cd -f values.yaml
```
---

# Example output of initial install as of doing it on 04/04/2026

NAME: argocd
LAST DEPLOYED: Sun Apr  5 00:43:32 2026
NAMESPACE: argocd
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
In order to access the server UI you have the following options:

1. kubectl port-forward service/argocd-server -n argocd 8080:443

    and then open the browser on http://localhost:8080 and accept the certificate

2. enable ingress in the values file `server.ingress.enabled` and either
      - Add the annotation for ssl passthrough: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-1-ssl-passthrough
      - Set the `configs.params."server.insecure"` in the values file and terminate SSL at your ingress: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-2-multiple-ingress-objects-and-hosts


After reaching the UI the first time you can login with username: admin and the random password generated during the installation. You can find the password by running:

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

(You should delete the initial secret afterwards as suggested by the Getting Started Guide: https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli)