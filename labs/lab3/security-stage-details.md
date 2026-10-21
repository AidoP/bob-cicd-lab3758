The `Security Analysis` stage you create in this lab should have the following structure:

---

**Environment Configuration:**
```groovy
environment {
    SONAR_TOKEN = credentials('sonarqube-token')
    SONAR_HOST_URL = 'https://sonarqube-sonarqube.apps.itz-y6rg5f.infra01-lb.wdc07.techzone.ibm.com'
}
```

---

**Stage Phases:**

_Phase 1: Mandatory SonarQube Security Scanning_

- **Runs SonarQube Scan** (required - must be configured with `SONAR_TOKEN` and `SONAR_PROJECT_KEY`)
   ```bash
   cd order-service && mvn clean compile sonar:sonar \
     -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
     -Dsonar.projectName="Order Service ${USER}" \
     -Dsonar.host.url=https://sonarqube-sonarqube.apps.itz-y6rg5f.infra01-lb.wdc07.techzone.ibm.com \
     -Dsonar.token=${SONAR_TOKEN} \
     -DskipTests
   ```
   - Uses the **PROJECT_KEY** and **TOKEN** created in Part 3
   - Skips tests to allow the scan to complete
   - Uploads analysis results to SonarQube

- **Fetches Issues from SonarQube API**
   - Retrieves all unresolved issues
   - Gets full details: rule, severity, location, message

- **Fetches Security Hotspots**
   - Retrieves security hotspots
   - Gets vulnerability probability ratings

- **Fetches Project Metrics**
   - Gets quality gate metrics and ratings
   - Retrieves bugs, vulnerabilities, code smells, coverage, technical debt

- **Generates Comprehensive Analysis Report**
   - Creates `SONARQUBE_ANALYSIS_REPORT.md` with:
     - Executive summary with quality gate status
     - Detailed breakdown of all issues
     - Security hotspots analysis with remediation code
     - Bugs and code smells with fixes
     - Remediation priority matrix
     - CI/CD integration recommendations

_Phase 2: Grype Vulnerability Scanning_
- **Installs Grype on-demand** in build-tools container
- Scans for vulnerabilities in dependencies and application code
- Counts Critical, High, and Medium severity CVEs
- Gracefully handles errors to prevent pipeline failure

_Phase 3: Intelligent CVE Analysis*_
- Checks if any vulnerabilities were found in reports
- **If vulnerabilities detected:** Runs detailed CVE analysis using Bob
  - Assesses severity and exploitability
  - Determines deployment impact
  - Provides remediation guidance
  - Generates `CVE_ANALYSIS_REPORT.md`
- **If no vulnerabilities:** Skips detailed analysis, creates clean report

_Phase 4: Risk Assessment & Deployment Decision_
- Calculates overall risk level from all reports: CRITICAL/HIGH/MEDIUM/LOW
- Makes deployment decision:
  - 🔴 **CRITICAL:** Blocks deployment (pipeline fails)
  - 🟠 **HIGH:** Marks pipeline unstable, requires review
  - 🟡 **MEDIUM:** Proceeds with caution
  - 🟢 **LOW:** Approved for deployment
- Generates `SECURITY_SUMMARY.md` with consolidated findings
- Archives all security artifacts

---

**Generated Reports:**
- `security-analysis-report.md` - Consolidated security report with executive summary
- `bob-cve-analysis.txt` - Bob's CVE analysis with remediation steps
- `order-service/grype-report.json` - Grype vulnerability scan results

---

**Security Risk Levels:**
- **🔴 CRITICAL:** Blocks deployment (pipeline fails)
- **🟠 HIGH:** Pipeline marked unstable, review required
- **🟡 MEDIUM:** Proceeds with caution, issues should be addressed soon
- **🟢 LOW:** Safe to deploy