# CVE_TABLE.md


Tout d’abord il faut scanner les dossiers du projet :
Il faut taper la commande suivante afficher sur la capture d’écran suivante :
```
trivy fs.
```

| Vulnérabilité | Référence (CVE/advisory) | Correctif appliqué (commande / version) | Gravité | OWASP Top-10 (2021) | Preuve |
|----------------|---------------------------|-----------------------------------------|----------|----------------------|---------|
| lodash: command injection | CVE-2021-23337 | npm install lodash@4.17.21 | HIGH | A06: Vulnerable and Outdated Components | ./evidence/trivy_before.json → ./evidence/trivy_after.json |
| node-forge: signature verification leniency | CVE-2022-24771 | npm install node-forge@1.3.0 | HIGH | A06: Vulnerable and Outdated Components | idem |
| serialize-javascript: code injection | CVE-2020-7660 | npm install serialize-javascript@3.1.0 | HIGH | A06: Vulnerable and Outdated Components | idem |
| Secret exposé (clé privée) | n/a | git rm --cached private-node.pem ; ajout à .gitignore | HIGH | A02: Cryptographic Failures | ./evidence/trivy_secret_before.json → ./evidence/trivy_secret_after.json |
```
Fichier scanné : private-node.pem
➡ Trivy a détecté une clé privée sensible dans ce fichier c'est Asymmetric Private Key```


| Secret exposé (clé privée) | n/a | git rm --cached private-node.pem ; ajout à .gitignore | HIGH | A02: Cryptographic Failures | ./evidence/gitleaks_after_proof.json |



| Vulnérabilité                | Référence (CVE/advisory)        | Correctif appliqué (commande / version)                  | Gravité (CVSS / label) | OWASP Top-10 (2021) | Preuve / fichier |
|-------------------------------|---------------------------------|----------------------------------------------------------|------------------------|--------------------|-----------------|
| lodash – Command Injection     | CVE-2021-23337                  | `npm install lodash@4.17.21`                             | HIGH                   | A1: Broken Access Control | `evidence/trivy_after.json` |
| lodash – ReDoS                 | CVE-2020-28500                  | `npm install lodash@4.17.21`                             | MEDIUM                 | A5: Security Misconfiguration | `evidence/trivy_after.json` |
| node-forge – Signature leniency| CVE-2022-24771                  | `npm install node-forge@1.3.0`                           | HIGH                   | A3: Cryptographic Failures | `evidence/trivy_after.json` |
| node-forge – Signature garbage | CVE-2022-24772                  | `npm install node-forge@1.3.0`                           | HIGH                   | A3: Cryptographic Failures | `evidence/trivy_after.json` |
| node-forge – Open Redirect     | CVE-2022-0122                   | `npm install node-forge@1.0.0`                           | MEDIUM                 | A7: Identification and Authentication Failures | `evidence/trivy_after.json` |
| node-forge – DigestInfo issue  | CVE-2022-24773                  | `npm install node-forge@1.3.0`                           | HIGH                   | A3: Cryptographic Failures | `evidence/trivy_after.json` |
| node-forge – Prototype Pollution| GHSA-5rrq-pxf6-6jx5             | `npm install node-forge@1.0.0`                           | LOW                    | A5: Security Misconfiguration | `evidence/trivy_after.json` |
| node-forge – URL parsing issue | GHSA-gf8q-jrpm-jvxq             | `npm install node-forge@1.0.0`                           | LOW                    | A5: Security Misconfiguration | `evidence/trivy_after.json` |
| serialize-javascript – Code Injection | CVE-2020-7660           | `npm install serialize-javascript@3.1.0`                | HIGH                   | A3: Injection       | `evidence/trivy_after.json` |
| serialize-javascript – XSS     | CVE-2019-16769                  | `npm install serialize-javascript@2.1.1`                | MEDIUM                 | A7: Identification and Authentication Failures | `evidence/trivy_after.json` |

Secrets supprimés

| Secret / fichier              | Action appliquée                    | Preuve / fichier |
|-------------------------------|-----------------------------------|-----------------|
| private-node.pem               | supprimé du suivi Git (`git rm --cached`) et ajouté à `.gitignore` | `evidence/gitleaks_final.json` |
| dossier `evidence/`           | ajouté à `.gitignore` pour éviter tout scan futur de secrets | `evidence/gitleaks_final.json` |

 Notes importantes
- Tous les secrets factices (clés, `.env`) ont été retirés et ne sont plus suivis par Git.  
- Le dossier `evidence/` contient les logs et fichiers JSON avant/après correction pour chaque scan (Trivy, Gitleaks).  
- Les scans Trivy et Gitleaks ont été relancés après correctifs pour valider la suppression des vulnérabilités et secrets.  
- Chaque vulnérabilité est associée à un correctif précis (commande et version de package).  
- OWASP Top-10 (2021) a été mappé pour chaque vulnérabilité détectée.  
- Pour chaque ligne du tableau, le fichier de preuve correspondant contient le log ou le scan montrant que la vulnérabilité a été corrigée.
```



 a) Suppression des secrets du suivi Git
```
git rm --cached private-node.pem || true
git rm --cached .env || true
```

b)Ignorer les fichiers et dossiers sensibles
```
  echo "private-node.pem" >> .gitignore
    echo ".env" >> .gitignore
   echo "evidence/" >> .gitignore
git add .gitignore
git commit -m "chore: remove tracked private key and ignore secrets and evidence folder"
```
c)vérification avec Trivy et Gitleaks

Avant correction : Trivy et Gitleaks détectaient private-node.pem comme HIGH risk.
Après suppression et mise à jour du .gitignore : plus de secrets suivis dans Git, et le dossier evidence/ est ignoré.
```
trivy fs --format table .
gitleaks detect --source . --report-path=./evidence/gitleaks_final.json
```
d)Push sur ma branche
```
git push origin fix/security-eden
```
POur SNYK:


````
# Installer Snyk
npm install -g snyk

# Se connecter
snyk auth
# Filtrer les failles HIGH et CRITICAL
snyk test --severity-threshold=high

# Exporter le rapport en JSON
snyk test --json > snyk-report.json
````

```
npm install serialize-javascript@6.0.2 --save
npm update serialize-javascript
```
J'ai repush ensuite et changer les modifs:
```
snyk test

Testing /Users/eden/SOUDRY/BTS_DEVSECOPS_CYBER_1...

Organization:      edenassant
Package manager:   npm
Target file:       package-lock.json
Project name:      vuln-node-project
Open source:       no
Project path:      /Users/eden/SOUDRY/BTS_DEVSECOPS_CYBER_1
Licenses:          enabled

✔ Tested 73 dependencies for known issues, no vulnerable paths found.

Next steps:
- Run `snyk monitor` to be notified about new related vulnerabilities.
- Run `snyk test` as part of your CI/test.
```

```
| Vulnérabilité              | Référence                                                                                                | Correctif appliqué                       | Gravité | OWASP Top-10 (2021) |
| -------------------------- | -------------------------------------------------------------------------------------------------------- | ---------------------------------------- | ------- | ------------------- |
| Cross-site Scripting (XSS) | [SNYK-JS-SERIALIZEJAVASCRIPT-6147607](https://security.snyk.io/vuln/SNYK-JS-SERIALIZEJAVASCRIPT-6147607) | `npm install serialize-javascript@6.0.2` | Medium  | A03 – Injection     |

```
Gitleaks
```
npm install gitleaks

added 1 package, and audited 75 packages in 515ms

14 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

Les secrets
```
https://github.com/edenassant/BTS_DEVSECOPS_CYBER_1/actions/runs/19075095065/attempts/1#summary-54488339792
```
![img.png](img.png)
<img width="2184" height="632" alt="image" src="https://github.com/user-attachments/assets/143b73ae-e7e2-43d3-9f00-43aeec2cb8b8" />


supp les secrets :
```
rm private-node.pem
rm -rf evidence/
```
