
# **Helm Repository via GitHub Pages – Setup & Update Guide**

This repository contains Helm charts hosted using **GitHub Pages**.
Follow this guide to build, package, index, publish, and update chart versions.

---

## **📌 1. Repository Structure**

```
helm/
 ├── nginx-app/
 │    ├── Chart.yaml
 │    ├── values.yaml
 │    ├── templates/
 ├── index.yaml       ← auto-generated
 ├── nginx-app-0.1.0.tgz
 ├── nginx-app-0.2.0.tgz
 └── README.md
```

---

## **📌 2. Initial Setup (One-Time Only)**

### **Enable GitHub Pages**

1. Go to GitHub repo → **Settings → Pages**
2. Set:

   * **Source:** `main`
   * **Folder:** `/root`
3. Save → GitHub publishes your repo at:

```
https://<username>.github.io/<repo>/
```

Example:

```
https://phaneendrareddyg.github.io/helm/
```

---

## **📌 3. Package a Chart (Every New Version)**

Go inside your Helm repo:

```bash
cd nginx-app
helm package .
```

This generates:

```
nginx-app-0.3.0.tgz
```

Move it to repo root:

```bash
mv nginx-app-0.3.0.tgz ..
cd ..
```

---

## **📌 4. Update Helm Repository Index**

Run:

```bash
helm repo index . --url https://phaneendrareddyg.github.io/helm
```

This updates/creates:

```
index.yaml
```

Commit & push:

```bash
git add .
git commit -m "Add version 0.3.0"
git push
```

---

## **📌 5. Verify Repository**

```bash
curl https://phaneendrareddyg.github.io/helm/index.yaml
```

OR

```bash
helm repo add myrepo https://phaneendrareddyg.github.io/helm
helm search repo myrepo
```

---

## **📌 6. Updating to a New Chart Version**

### **Step 1 — Update Chart.yaml**

Inside `nginx-app/Chart.yaml`

```yaml
version: 0.3.0
appVersion: "1.26.0"
```

### **Step 2 — Package the chart**

```bash
helm package nginx-app/
```

### **Step 3 — Update index**

```bash
helm repo index . --url https://phaneendrareddyg.github.io/helm
```

### **Step 4 — Commit & push**

```bash
git add .
git commit -m "Release nginx-app 0.3.0"
git push
```

---

## **📌 7. Use the Chart in ArgoCD**

Example ArgoCD Application:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-app
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://phaneendrareddyg.github.io/helm
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

## **📌 8. Common Errors & Fixes**

### **❌ index.yaml not found (404)**

Your repo folder must contain:

```
index.yaml
nginx-app-0.x.x.tgz
```

### **❌ Chart.yaml file is missing**

You packaged incorrectly — ensure directory structure:

```
nginx-app/
    Chart.yaml
    templates/
```

Run:

```bash
helm lint nginx-app
```

### **❌ ArgoCD cannot load chart**

Fix by updating index file:

```bash
helm repo index . --url https://phaneendrareddyg.github.io/helm
```

