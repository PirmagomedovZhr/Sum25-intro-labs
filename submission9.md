# Lab 9 — DevSecOps: Vulnerability Scanning

## 🔍 Task 1: Dynamic Application Security Testing (DAST)

### 1.1 Environment Setup  
Create an isolated Docker bridge network so ZAP can resolve the target container by name:

```bash
docker network create zapnet
```

Run the vulnerable OWASP Juice Shop application:

```bash
docker run -d --name juice-shop --network zapnet -p 3000:3000   bkimminich/juice-shop
```

Run the OWASP ZAP Baseline (passive) scan and export the report to HTML:

```bash
docker run --rm --network zapnet   -u zap -v "$PWD:/zap/wrk"   ghcr.io/zaproxy/zaproxy:stable   zap-baseline.py -t http://juice-shop:3000 -r zap-report.html
```

Report generated: `zap-report.html`

### 1.2 ZAP Scan Results  

| Metric | Value |
| ------ | ----- |
| URLs crawled | 95 |
| Passed checks | 56 |
| New warnings | 10 |
| New failures | 0 |

**Top warnings**

| Risk | Alert | Count | Recommendation |
| ---- | ----- | ----- | -------------- |
| Medium | Content‑Security‑Policy header not set | 18 | Add strict CSP (`default-src 'self'`) |
| Medium | Cross‑Domain Misconfiguration (CORS) | 11 | Restrict `Access-Control-Allow-Origin` |
| Low | Cross‑Domain JS Source Inclusion | 22 | Host JS locally / use SRI |
| Low | Deprecated `Feature‑Policy` header | 12 | Replace with `Permissions‑Policy` |
| Low | Timestamp Disclosure (Unix) | 13 | Remove build/version hints |
| Low | Dangerous JS Functions | 2 | Remove `eval`, `document.write`, etc. |

---

## 🐳 Task 2: Container & Dependency Scanning (Trivy)

### 2.1 Running Trivy

```bash
# Pull image (if missing)
docker pull bkimminich/juice-shop:latest

# Scan only HIGH / CRITICAL issues
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock   aquasec/trivy:latest image   --severity HIGH,CRITICAL bkimminich/juice-shop
```

Log saved: `lab9_output.txt`

### 2.2 Trivy Summary  

| Severity | Count | Example CVE / Package |
| -------- | ----- | --------------------- |
| Critical | 0 | — |
| High | 1 | `base64url` — CVE‑2022‑37012 (DoS) |

Trivy examined 132 Debian packages and 9 npm manifests; only one High CVE, no Critical.

---

## 🛠 Recommended Remediations

* **Add HTTP headers:** `Content‑Security‑Policy`, `Permissions‑Policy`, `X‑Frame‑Options`, etc.  
* **Tighten CORS:** allow only trusted origins, avoid wildcards.  
* **Update dependencies:** upgrade/remove vulnerable `base64url`; run `npm audit fix`.  
* **Refresh base image:** rebuild on `node:20‑alpine` (current LTS).  
* **Automate scans in CI/CD:** integrate ZAP baseline & Trivy; fail pipeline on new High/Critical findings.  

---

## ✅ Conclusions

* OWASP ZAP revealed medium/low‑risk misconfigurations (missing CSP, lax CORS, legacy headers).  
* Trivy detected a single High vulnerability in an npm dependency; no Critical issues.  
* Applying the recommended fixes and automating both scans will significantly improve the security posture before production deployments.  

---

Prepared by: Zhalil — July 2025
