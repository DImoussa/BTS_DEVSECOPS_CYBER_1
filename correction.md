## 🔐 Corrections appliquées dans le projet

### ✅ 1. Mise à jour des dépendances

Mise à jour de sécurité dans `package.json` et `package-lock.json` :

- `serialize-javascript` : **2.1.0 → 6.0.2** ✅ (correction XSS critique)
- Propositions de mises à jour *(à appliquer)* :
    - `lodash` → **4.17.21**
    - `node-forge` → **1.3.1**

---

### 🛠️ 2. Modifications du code source

Suppression de l’option `unsafe: true` dans la sérialisation pour éviter les XSS :

```js
// Avant
serialize(obj, { unsafe: true })

// Après
serialize(obj)
```

Recommandation : éviter la compilation des templates Lodash côté client  
(pour prévenir l'exécution de code arbitraire — RCE).

---

### 🔑 3. Gestion des secrets

Clé privée SSH retirée du dépôt :

```bash
git rm --cached private-node.pem
echo "private-node.pem" >> .gitignore
```

✅ Recommandation : utilisation d’un gestionnaire de secrets sécurisé.

---

## 🧩 Commandes utilisées / recommandées

```bash
npm install serialize-javascript@6.0.2
npm install lodash@4.17.21
npm install node-forge@1.3.1

git rm --cached private-node.pem
echo "private-node.pem" >> .gitignore
git commit -m "Retirer clé privée et mise à jour du .gitignore"

npm audit
npm audit fix --force

git add package.json package-lock.json
git commit -m "Mise à jour des dépendances pour correction vulnérabilités"
git push origin main

snyk code test --sarif    # Test local Snyk (optionnel)
```

---

## 🔍 Workflows GitHub Actions configurés

### 1️⃣ Simple scan `npm audit`

```yaml
name: Simple npm audit

on: [push, pull_request]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install dependencies
        run: npm install
      - name: Run npm audit
        run: npm audit --audit-level=moderate
```

---

### 2️⃣ Gitleaks Quick Scan

```yaml
name: Gitleaks Quick Scan

on: [push, pull_request]

jobs:
  gitleaks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Gitleaks
        uses: zricethezav/gitleaks-action@v2.14.0
```

---

### 3️⃣ Trivy Scan

```yaml
name: Trivy Scan

on: [push, pull_request]

jobs:
  trivy_scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: fs
          severity: HIGH,CRITICAL
```

---

### 4️⃣ Snyk Security (correctif simplifié, sans build Docker)

```yaml
name: Snyk Security

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

permissions:
  contents: read

jobs:
  snyk:
    permissions:
      contents: read
      security-events: write
      actions: read
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Snyk CLI to check for security issues
        uses: snyk/actions/setup@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      - name: Test for vulnerabilities
        run: snyk test --severity-threshold=high
```
---

## 🔎 Commandes complètes de scans de sécurité

### 🧪 NPM Audit

```bash
npm audit
npm audit fix --force
```

---

### 🛡️ Snyk (SAST local)

Commande locale simple pour scanner le code :

```bash
snyk code test --sarif
```

---

### 🔍 Gitleaks

Le plus souvent utilisé via GitHub Actions.  
Pour un scan local :

```bash
gitleaks detect
```

*(Sur GitHub Actions, aucune commande shell — juste la configuration YAML)*

---

### 🧱 Trivy

Scan local du système de fichiers :

```bash
trivy fs --severity HIGH,CRITICAL .
```

*(Sur GitHub Actions, appelé via l’action suivante)*

```yaml
uses: aquasecurity/trivy-action@master
with:
  scan-type: fs
  severity: HIGH,CRITICAL
```

---

## 📌 Résumé des workflows associés

| Outil | Objectif | Où s'exécute-t-il ? |
|-------|----------|-------------------|
| **npm audit** | Scan des dépendances npm | CI & local |
| **Snyk** | Analyse vulnérabilités + code (SAST) | CI & local |
| **Gitleaks** | Détection de secrets exposés | CI & local |
| **Trivy** | Scan vulnérabilités système et fichiers | CI & local |

---
