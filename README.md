# 🛡️ AppSec CI/CD Pipeline with Trivy and DefectDojo

This project demonstrates a hands-on Application Security (AppSec) pipeline built for continuous integration (CI/CD) that includes:

- Vulnerable target: OWASP Juice Shop
- Security scanner:
       - Trivy – vulnerability scanning for containers and filesystems
       - Semgrep – static application security testing (SAST) and code quality analysis
- Reporting dashboard: DefectDojo
- Pipeline orchestration: GitHub Actions

## 📌 Objectives

- Automate security scanning in a CI/CD pipeline
- Detect real-world vulnerabilities in a vulnerable application and code quality issues  
- Import scan results into DefectDojo for centralized reporting and management
- Generate clear, actionable vulnerability and code analysis reports

---

## 🏗️ Project Structure
.                                                                                                           
├── .github/                                                                                                                                                                                                                                             
│   └── workflows/                                                                                                                                                                                                                                                               
│       └── ci-cd-pipeline.yml                                                                  
├── Finding Report Semgrep P1.pdf   
├── Finding Report Semgrep P2.pdf  
├── Finding ReportTrivy.pdf  
└── README.md                                                                                                         
├── Trivy-report.json  
├── semgrep-report.json 

---

## ⚙️ Tools & Technologies

| Tool         | Purpose                            |
|--------------|------------------------------------|
| GitHub Actions | CI/CD pipeline orchestration      |
| Trivy        | Security scanning (filesystem, image) |
| Semgrep        | Static Application Security Testing (SAST) |
| DefectDojo   | Vulnerability management & reporting |
| OWASP Juice Shop | Vulnerable target for AppSec testing |

---

## 🚀 How to Run This Project

### 1. Clone the Repository

```
git clone https://github.com/yourusername/appsec-ci-cd-pipeline.git
cd appsec-ci-cd-pipeline    
```

### 2. Start DefectDojo (Docker)
   
```
git clone https://github.com/DefectDojo/django-DefectDojo.git
cd django-DefectDojo
docker-compose up -d
```
Wait until DefectDojo UI becomes available at: http://localhost:8080

Default login:
```
Username: admin
Password: admin
```

### 3. Run Trivy Scan

Example for Docker image:
```
trivy image bkimminich/juice-shop --format json --output trivy-report.json
```
OR for local folder:
```
trivy fs ./juice-shop --format json --output trivy-report.json
```

### 4. Run Semgrep Scan

Run Semgrep container to scan the repository source code:
```
docker run --rm -v $(pwd):/src returntocorp/semgrep semgrep scan --config=auto --json --output=semgrep-report.json
```

### 5. Upload Scan to DefectDojo (Manually)

Go to:

- Engagements > Import Scan
- Select:
    - Scan Type: Trivy
    - File: Trivy-report.json or semgrep-report.json
    - Product Name: Juice Shop AppSec
    - Engagement Name: CI/CD Pipeline Scan
---

### 🤖 GitHub Actions CI/CD Workflow

```
name: AppSec CI/CD Security Pipeline

on:
  push:
    branches:
      - main

jobs:
  security-scan:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Install Trivy
      run: |
        curl -sfL https://github.com/aquasecurity/trivy/releases/download/v0.38.3/trivy_0.38.3_Linux-64bit.deb -o trivy.deb
        sudo dpkg -i trivy.deb

    - name: Run Trivy scan
      run: |
        trivy fs . --format json --output Trivy-report.json

    - name: Upload report to GitHub
      uses: actions/upload-artifact@v4
      with:
        name: trivy-report
        path: Trivy-report.json
    - name: Run Semgrep scan
      run: |
        docker run --rm -v ${{ github.workspace }}:/src returntocorp/semgrep semgrep scan --config=auto --json --output=semgrep-report.json

    - name: Upload Semgrep report
      uses: actions/upload-artifact@v4
      with:
        name: semgrep-report
        path: semgrep-report.json

    - name: Upload Trivy report to DefectDojo
      env:
        DEFECTDOJO_URL: ${{ secrets.DEFECTDOJO_URL }}
        DEFECTDOJO_USERNAME: ${{ secrets.DEFECTDOJO_USERNAME }}
        DEFECTDOJO_PASSWORD: ${{ secrets.DEFECTDOJO_PASSWORD }}
      run: |
        curl -X POST \
          -F "file=@Trivy-report.json" \
          -F "product_name=Juice Shop AppSec" \
          -F "engagement_name=CI/CD Pipeline Scan" \
          -F "scan_type=Trivy" \
          -u "${DEFECTDOJO_USERNAME}:${DEFECTDOJO_PASSWORD}" \
          "${DEFECTDOJO_URL}/api/v2/import-scan/"

    - name: Upload Semgrep report to DefectDojo
      env:
        DEFECTDOJO_URL: ${{ secrets.DEFECTDOJO_URL }}
        DEFECTDOJO_USERNAME: ${{ secrets.DEFECTDOJO_USERNAME }}
        DEFECTDOJO_PASSWORD: ${{ secrets.DEFECTDOJO_PASSWORD }}
      run: |
        curl -X POST \
          -F "file=@semgrep-report.json" \
          -F "product_name=Juice Shop AppSec" \
          -F "engagement_name=CI/CD Pipeline Scan" \
          -F "scan_type=Semgrep JSON Report" \
          -u "${DEFECTDOJO_USERNAME}:${DEFECTDOJO_PASSWORD}" \
          "${DEFECTDOJO_URL}/api/v2/import-scan/"
```
--- 

### 🧾 Sample Finding

- Vulnerability ID: CVE-2023-46233
- Affected Package: crypto-js
- Version: 3.3.0
- Severity: Critical
- File Path: juice-shop/node_modules/crypto-js/package.json
- Fixed Version: >= 4.2.0

#### 📄 Description:

  PBKDF2 implementation uses SHA1 and 1 iteration by default. Vulnerable to brute-force attacks.

---

### 📸 Screenshots

- DefectDojo Report UI – Trivy scan Report

  ![image](https://github.com/user-attachments/assets/ef11c1de-3c46-4c79-9d4d-da5dd665362c)

- CI/CD run – GitHub Actions output

  ![image](https://github.com/user-attachments/assets/fe432311-d5ab-428e-be21-90443b01a979)

- Example report

  ![image](https://github.com/user-attachments/assets/b2b3d7a1-e427-472e-80bf-04b1db6555e5)

---

### 🧠 Lessons Learned

- Integrating Trivy into CI/CD is simple but powerful.
- DefectDojo provides a strong base for vulnerability management.
- Real-world vulnerable apps (like Juice Shop) make security testing meaningful.
