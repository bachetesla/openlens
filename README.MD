Below is a polished, concise, and user-friendly `README.md` for your GitHub repository, including instructions for retrieving tokens and adding them to `kubeconfig`. Following that, I'll provide the full code for both manifest files.

---

### README.md

```markdown
# Kubernetes Admin & Viewer Access

Welcome to this Kubernetes setup! This repository provides two manifest files to create ServiceAccounts with distinct roles in your cluster:
- **`k8s-admin-manifest.yaml`**: Grants full `cluster-admin` privileges for unrestricted access.
- **`k8s-viewer-manifest.yaml`**: Grants `view` privileges for read-only access.

Whether you need to manage your cluster or just peek inside, these manifests have you covered.

---

## What's Inside?

Each manifest creates:
- A `Secret` to hold the ServiceAccount token.
- A `ServiceAccount` for authentication.
- A `ClusterRoleBinding` to assign the role (`cluster-admin` or `view`).

- **Admin**: Full control over the cluster.
- **Viewer**: Read-only access (no secrets or modifications).

---

## Prerequisites

- A Kubernetes cluster (v1.18+ recommended).
- `kubectl` installed and configured.
- RBAC enabled.

---

## Get Started

1. **Clone the repo**:
   ```bash
   git clone https://github.com/<your-username>/<your-repo>.git
   cd <your-repo>
   ```

2. **Deploy the manifests**:
   - For admin access:
     ```bash
     kubectl apply -f k8s-admin-manifest.yaml
     ```
   - For viewer access:
     ```bash
     kubectl apply -f k8s-viewer-manifest.yaml
     ```

3. **Check everything’s there**:
   ```bash
   kubectl get secret admin-secret viewer-secret -n default
   kubectl get sa admin-service-account viewer-service-account -n default
   kubectl get clusterrolebinding admin-cluster-binding viewer-cluster-binding
   ```

---

## Grab the Tokens & Configure Access

### Admin Token
1. **Extract the token**:
   ```bash
   TOKEN=$(kubectl get secret admin-secret -n default -o jsonpath='{.data.token}' | base64 -d)
   echo $TOKEN
   ```
2. **Add it to kubeconfig**:
   ```bash
   kubectl config set-credentials admin-user --token=$TOKEN
   kubectl config set-context admin-context --cluster=<cluster-name> --user=admin-user
   kubectl config use-context admin-context
   ```
3. **Test it**:
   ```bash
   kubectl get nodes
   ```

### Viewer Token
1. **Extract the token**:
   ```bash
   TOKEN=$(kubectl get secret viewer-secret -n default -o jsonpath='{.data.token}' | base64 -d)
   echo $TOKEN
   ```
2. **Add it to kubeconfig**:
   ```bash
   kubectl config set-credentials viewer-user --token=$TOKEN
   kubectl config set-context viewer-context --cluster=<cluster-name> --user=viewer-user
   kubectl config use-context viewer-context
   ```
3. **Test it**:
   ```bash
   kubectl get pods --all-namespaces  # Works!
   kubectl delete pod <pod-name>     # Fails (read-only)
   ```

Replace `<cluster-name>` with your cluster’s name (check with `kubectl config get-clusters`).

---

## Example kubeconfig

Here’s how your `~/.kube/config` might look after setup:
```yaml
apiVersion: v1
clusters:
- cluster:
    server: https://<cluster-api-server>
  name: <cluster-name>
contexts:
- context:
    cluster: <cluster-name>
    user: admin-user
  name: admin-context
- context:
    cluster: <cluster-name>
    user: viewer-user
  name: viewer-context
users:
- name: admin-user
  user:
    token: <admin-token>
- name: viewer-user
  user:
    token: <viewer-token>
```

---

## Security Tips

- **Admin Access**: Full power—keep it locked down.
- **Viewer Access**: Safe for monitoring, but no secrets access.
- **Tokens**: Treat them like passwords—don’t share or commit them.
- **Namespace**: Using `default` here; tweak to your needs.
- **Rotation**: Recreate ServiceAccounts now and then for fresh tokens.

---

## Cleanup

Done? Tear it down:
```bash
kubectl delete -f k8s-admin-manifest.yaml
kubectl delete -f k8s-viewer-manifest.yaml
```

---

## Troubleshooting

- **Missing resources?** Check the controller manager (`kubectl get pods -n kube-system`).
- **Access denied?** Ensure RBAC is on and manifests applied correctly.
- **Token not working?** Inspect the Secret (`kubectl describe secret <name> -n default`).
- **Context confusion?** Peek at your config (`kubectl config view`).

---

## Let’s Make It Better

Got ideas or fixes? Open an issue or send a pull request—I’d love to hear from you!

---

## License

This is under the [MIT License](LICENSE)—feel free to use and adapt it.
```

---

### Full Code

#### `k8s-admin-manifest.yaml`
```yaml
---
# Secret for Cluster Admin ServiceAccount token
apiVersion: v1
kind: Secret
metadata:
  name: admin-secret
  namespace: default
  annotations:
    kubernetes.io/service-account.name: admin-service-account
type: kubernetes.io/service-account-token

---
# ServiceAccount for Cluster Admin
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-service-account
  namespace: default
secrets:
  - name: admin-secret

---
# ClusterRoleBinding to grant cluster-admin role
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-cluster-binding
subjects:
  - kind: ServiceAccount
    name: admin-service-account
    namespace: default
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

#### `k8s-viewer-manifest.yaml`
```yaml
---
# Secret for Viewer ServiceAccount token
apiVersion: v1
kind: Secret
metadata:
  name: viewer-secret
  namespace: default
  annotations:
    kubernetes.io/service-account.name: viewer-service-account
type: kubernetes.io/service-account-token

---
# ServiceAccount for Viewer
apiVersion: v1
kind: ServiceAccount
metadata:
  name: viewer-service-account
  namespace: default
secrets:
  - name: viewer-secret

---
# ClusterRoleBinding to grant view role
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: viewer-cluster-binding
subjects:
  - kind: ServiceAccount
    name: viewer-service-account
    namespace: default
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io
```

