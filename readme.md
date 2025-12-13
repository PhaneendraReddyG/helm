
# **Helm Repository via GitHub Pages – Setup & Update Guide**

This repository contains Helm charts published via **GitHub Pages** using the `docs/` folder. Follow this guide to build, package, index, publish, and update chart versions.

---

## **📌 1. Repository Structure**

```
helm/
 ├── docs/
 │    ├── index.yaml
 │    ├── nginx-app-0.1.0.tgz
 │    └── nginx-app-0.2.0.tgz
 ├── nginx-app/
 │    ├── Chart.yaml
 │    ├── values.yaml
 │    └── templates/
 └── readme.md
```

---

## **📌 2. GitHub Pages (One-Time Setup)**

- In the GitHub repository: **Settings → Pages**
- Set **Branch** to `main` and **Folder** to `/docs` (or update examples below if you prefer publishing from root).
- Your published URL will be:

```
https://<your-username>.github.io/<repo>/
```

Replace `<your-username>` and `<repo>` with your GitHub username and repository name.

---

## **📌 3. Package a Chart (Every New Version)**

- From the repository root:

```powershell
cd nginx-app
helm package .
```

- Move the generated package into the `docs/` folder:

```powershell
Move-Item .\nginx-app-*.tgz ..\docs\
```

---

## **📌 4. Update Helm Repository Index**

- From repository root (recommended explicit path):

```powershell
helm repo index docs --url https://<your-username>.github.io/<repo>
```

- Or change into `docs/` and run:

```powershell
cd docs
helm repo index . --url https://<your-username>.github.io/<repo>
```

---

## **📌 5. Commit & Push**

- From repository root:

```powershell
git add docs/*
git commit -m "chore(release): add nginx-app x.y.z"
git push
```

---

## **📌 6. Verify Repository**

- Check that `index.yaml` is reachable:

```powershell
curl https://<your-username>.github.io/<repo>/index.yaml
```

- Or add the repo locally with Helm and search:

```powershell
helm repo add myrepo https://<your-username>.github.io/<repo>
helm search repo myrepo
```

---

## **📌 7. Updating to a New Chart Version (Workflow)**

1. Update `version:` in `nginx-app/Chart.yaml`.
2. Package the chart:

```powershell
cd nginx-app
helm package .
```

3. Move package to `docs/` and update index:

```powershell
Move-Item .\nginx-app-*.tgz ..\docs\
helm repo index docs --url https://<your-username>.github.io/<repo>
```

4. Commit and push:

```powershell
git add docs/*
git commit -m "release: nginx-app x.y.z"
git push
```

---

## **📌 8. Use the Chart in ArgoCD**

Example ArgoCD Application (replace placeholders):

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://<your-username>.github.io/<repo>
    chart: nginx-app
    targetRevision: "0.3.0"
    helm:
      releaseName: nginx-app
  destination:
    server: https://kubernetes.default.svc
    namespace: nginx-helm
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## **📌 9. Common Errors & Fixes**

- **index.yaml 404**: Ensure `docs/index.yaml` and the chart `*.tgz` files are present in the `docs/` folder and Pages is configured to publish `/docs` from `main`.
- **Chart.yaml missing**: Make sure your chart directory contains `Chart.yaml` and `templates/`. Run:

```powershell
helm lint nginx-app
```

- **ArgoCD cannot load chart**: Re-run `helm repo index docs --url https://<your-username>.github.io/<repo>`, commit `docs/index.yaml`, and push.


