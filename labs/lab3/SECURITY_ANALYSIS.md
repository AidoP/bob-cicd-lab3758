# Lab - Security Vulnerability Analysis

## Table of Contents

- [Overview](#overview)
- [Before you start](#before-you-start)
- [Part 1: Inject Security Vulnerabilities](#part-1-inject-security-vulnerabilities)
- [Part 2: Security Analysis with Software Security Reviewer Mode](#part-2-security-analysis-with-software-security-reviewer-mode)
- [Part 3: SonarQube Security Scanning](#part-3-sonarqube-security-scanning)
- [Part 4: Add SonarQube & Grype Security Analysis Stage](#part-4-add-sonarqube--grype-security-analysis-stage)
- [Part 5: Push and Watch](#part-5-push-and-watch)
- [Part 6: Restore Original Code](#part-6-restore-original-code)
- [Lab Summary](#lab-summary)
- [Troubleshooting](#troubleshooting)
- [Additional Information and Resources](#additional-information-and-resources)

## Overview

In this lab, you will learn how to perform security analysis with Bob Findings, run automated scans with SonarQube, and integrate comprehensive security checks into a CI/CD pipeline.

You'll inject security vulnerabilities into the order-service, analyze them with Bob's real-time findings and a comprehensive security review mode, run automated SonarQube scans, integrate security checks into your Jenkins pipeline, and learn proper Git workflows for managing experimental changes.

### What you'll build / test

- **Security vulnerability injection** — use a script to inject 6 types of vulnerabilities into OrderService.java and observe Bob's real-time findings as they appear in your IDE.
- **Comprehensive security analysis** — import and use the Software Security Reviewer mode to generate a full application security assessment covering code, infrastructure, compliance, and remediation roadmap.
- **SonarQube integration** — generate a SonarQube token, run automated security scans, and review detailed vulnerability reports with metrics and issue tracking.
- **Enhanced Jenkins pipeline** — add a comprehensive security stage to your Jenkinsfile that runs SonarQube scans and generates security reports on every build.
- **Git workflow best practices** — use `git restore` to discard experimental vulnerability injections and maintain a clean working directory before pushing changes.

By the end, every push triggers automated security scanning in your Jenkins pipeline, and you've practiced using Bob's security analysis capabilities across multiple tools and workflows, including proper Git hygiene for managing temporary code changes.


## Before you start

- [ ] Lab 1 complete (askBob helper added to your Jenkinsfile)
- [ ] You're on your working branch (e.g. `user1-labs`)
- [ ] Your instructor will provide you the credentials for SonarQube

**Reset your Jenkinsfile first.** Remove any stages you added in previous labs, keeping only the `Checkout` stage and the `askBob` helper at the bottom. Stacking every lab's stages stretches a single build past 15 minutes — clearing them keeps iteration fast and the logs readable. Use [`solution/Jenkinsfile.start`](../../solution/Jenkinsfile.start) as the reference shape.

---

## Part 1: Inject Security Vulnerabilities

1. Lets first review the `Bob Findings` before injecting some vulnerabilities

1. Select **Bob Findings** at the bottom of the screen. You will observe that it is currently empty. This is expected, as no vulnerabilities have been injected into the file yet, so Bob has no findings to display.

1. For the purposes of this lab, we will inject some vulnerabilities into our sample application.

1. Ensure you are in **Agent Mode** before prompting Bob with the following:

      ```
      Bob run script inject_vulnerabilities.sh
      ```
1. Running the script:

   - Modifies `order-service/src/main/java/com/example/orders/service/OrderService.java`
   - Injects vulnerable dependencies into `order-service/pom.xml`
   - Injects 9 types of vulnerabilities:
   - **Code Vulnerabilities (6):**
      - Hardcoded credentials (BACKUP_DB_PASSWORD, LEGACY_API_KEY)
      - Insecure logging (System.out.println)
      - Weak cryptography (MD5)
      - Stack trace exposure (printStackTrace)
      - Weak random (java.util.Random)
      - Information exposure (detailed error messages)
   - **Dependency Vulnerabilities (3):**
      - Log4j 2.14.1 - CVE-2021-44228 (Log4Shell - Critical)
      - Jackson Databind 2.12.3 - CVE-2020-36518 (High severity)
      - SnakeYAML 1.30 - CVE-2022-1471 (Critical severity)

1. The three vulnerable dependencies introduce **multiple high and critical severity CVEs** that will be detected by Grype during the security scan section of the lab. This will be used to demonstrate how dependency vulnerabilities are identified, assessed, and prioritized in the pipeline.

1. The **Expected Output** of the script should be similar to the following:
   ```
   🔧 Vulnerability Injection Script
   ==================================

   🔧 Injecting vulnerabilities into OrderService.java...
      ✅ Java vulnerabilities injected

   🔧 Injecting vulnerable Log4j dependency into pom.xml...
      ✅ Vulnerable Log4j 2.14.1 dependency injected (CVE-2021-44228)

   ✅ Vulnerability injection complete!

   📍 Modified files:
      - order-service/src/main/java/com/example/orders/service/OrderService.java
      - order-service/pom.xml

   🔍 Injected vulnerabilities:
      1. Hardcoded credentials (BACKUP_DB_PASSWORD, LEGACY_API_KEY)
      2. Insecure logging (System.out.println with sensitive data)
      3. Weak cryptography (MD5 in generateOrderVerificationCode)
      4. Stack trace exposure (printStackTrace in multiple methods)
      5. Weak random (java.util.Random in generateTrackingNumber)
      6. Information exposure (detailed error messages)
      7. Vulnerable dependency (Log4j 2.14.1 - CVE-2021-44228)
      8. Vulnerable dependency (Jackson Databind 2.12.3 - CVE-2020-36518)
      9. Vulnerable dependency (SnakeYAML 1.30 - CVE-2022-1471)
   ```

1. Now that vulnerabilities have been injected, let's see what Bob detects in real-time. **Open the modified file:**
   - Navigate to: `order-service/src/main/java/com/example/orders/service/OrderService.java`

1. Click **"Bob Findings"** at the bottom of the screen.
   - Bob will analyze the file and identify security issues in real-time
   - **You should see 4 critical findings** displayed in the Bob Findings panel

   > **Note:** Bob Findings should update with its security findings, if it does not, try to close and reopen the `OrderService.java` file or restart Bob

1. Bob Findings intelligently prioritizes and displays the **most critical security issues** that require immediate attention. We expect to find 4 Critical Findings in OrderService.java
   - 🔴 **Scan (Secrets): Secret Keyword:19** - Hardcoded credential detected
   - 🔴 **Scan (Secrets): Secret Keyword:20** - Hardcoded credential detected
   - 🔴 **Scan (Vulnerabilities): Integer.toHexString...** - Weak cryptography implementation
   - 🔴 **Scan (Vulnerabilities): Detected MD5 hash...** - Insecure hash algorithm usage

      > _**Note: Why 4 findings instead of 9 ?**_ While the script injected 9 types of vulnerabilities, Bob Findings focuses on **critical security issues** that pose the highest risk:
      > 
      >| Vulnerability Type | Detected by Bob Findings? | Reason |
      >|-------------------|---------------------------|---------|
      >| Hardcoded credentials | ✅ Yes (2 findings) | **CRITICAL** - Direct security breach risk |
      >| Weak cryptography (MD5) | ✅ Yes (2 findings) | **CRITICAL** - Cryptographically broken algorithm |
      >| Insecure logging (System.out) | ⚠️ Lower priority | Code smell, not critical vulnerability |
      >| Stack trace exposure | ⚠️ Lower priority | Context-dependent risk |
      >| Weak random (java.util.Random) | ⚠️ Lower priority | May be flagged differently |
      >| Information disclosure | ⚠️ Lower priority | Context-dependent risk |
      >| Vulnerable dependencies (3) | ⚠️ Not in code scan | Detected by dependency scanners (SonarQube/Grype) |
      >
      > **This is expected behavior!** Bob Findings prioritizes critical security issues (secrets and cryptographic vulnerabilities) over code quality issues. The comprehensive Security Analysis Report in Part 2 will capture all issues, while Bob Findings focuses on what developers need to fix immediately.

1. This gives you a preview of what Bob detects at the code level before we perform a comprehensive security analysis in Part 2.


## Part 2: Security Analysis with Software Security Reviewer Mode

Before importing the `Software Security Reviewer` mode, take a moment to understand what it does and how it differs from Bob Findings.

1. Ask Bob to explain. In `Ask` mode, prompt Bob with the following:

   ```
   Read @software-security-reviewer.yaml and explain what the Software Security Reviewer mode does and how it differs from Bob Findings. What types of analysis does it perform?
   ```

1. Bob should return an explanation that covers the following:
   - **Software Security Reviewer** is a specialized mode for comprehensive application security audits
   - It goes beyond code-level analysis to include infrastructure, configuration, and compliance
   - Performs multi-layer security assessment across the entire application stack
   - Generates detailed reports with remediation roadmaps and compliance mapping

1. Let's import the specialized **Software Security Reviewer** mode. Click the **Settings icon** (⚙️) in the top right corner of Bob

1. Click **"Modes"** in the settings menu, then Click the **"Import"** button

1. Select the file: `labs/lab3/software-security-reviewer.yaml` and Click **"Import"** to confirm.

1. Verify the mode was imported. You should see **"🛡️🔐 Software Security Reviewer"** in your modes list

1. Close the settings panel

1. Select the **"🛡️🔐 Software Security Reviewer"**  mode from the dropdown

1. Now let's have Bob perform a full security audit of the order-service application. Prompt Bob with the following: 

   ```
   Evaluate order-service for vulnerabilities, insecure patterns, misconfigurations, and compliance gaps and generate an analysis report.
   ```

1. The approach Bob takes may vary. Sometimes Bob creates a TODO list to track the analysis process, other times it proceeds directly to scanning. Both approaches are valid. In either case, Bob will:

   - Scans all application files (Java code, configuration, Dockerfile, K8s manifests)
   - Identifies security vulnerabilities and insecure patterns
   - Analyzes compliance gaps (PCI DSS, OWASP Top 10)
   - Evaluates misconfigurations in infrastructure
   - Generates comprehensive security report: `SECURITY_ANALYSIS_REPORT.md`

1. Review the `SECURITY_ANALYSIS_REPORT.md` report to understand each finding, its impact, and remediation steps. It should include:

   - Detailed findings with CVSS scores, CWE mappings, and code examples
   - Specific remediation steps for each vulnerability
   - OWASP Top 10 (2021) compliance matrix
   - PCI-DSS compliance status
   - Prioritized remediation roadmap (4 phases over 6 weeks)
   - Supply chain & infrastructure security recommendations
   - CI/CD pipeline security gates

1. Between the Bob Findings and this Software Security Reviewer mode we now have a comprehensive security assessment that together provides:
   - 🔍 **Real-time detection** (Bob Findings: 4 critical items) + **Comprehensive analysis** (Security Reviewer: 17 total findings)
   - 💻 **Critical code issues** + **Full code & configuration security**
   - ⚡ **Developer feedback** + **Security team reporting**
   - 🎯 **Immediate fixes** + **Strategic remediation planning**


## Part 3: SonarQube Security Scanning

1. Click **"Start New Task"** in Bob to begin a fresh conversation. This ensures Bob focuses on the SonarQube scanning workflow without context from the previous security analysis.

1. Switch to **Agent Mode** before proceeding with this step.

1. Prompt Bob to create a SonarQube project and token: 
   > **⚠️ IMPORTANT:** Before executing the command below, replace `${USER}` with your actual username (e.g., `user1`, `user2`, etc.) in the prompt. This ensures your SonarQube project is uniquely identified. Also replace `{REPLACE_SONARQUBE_PASSWORD}` with the credential provided by the instructor.

   ```
   Execute the following to create a SonarQube token and project for ${USER}:

   TOKEN=$(curl -s -u txcdemo:{REPLACE_SONARQUBE_PASSWORD} -X POST "https://sonarqube-sonarqube.apps.itz-y6rg5f.infra01-lb.wdc07.techzone.ibm.com/api/user_tokens/generate?name=token-$(date +%s)" | jq -r .token); PROJECT_KEY=order-service-${USER}-$(date +%s); curl -u txcdemo:{REPLACE_SONARQUBE_PASSWORD} -X POST "https://sonarqube-sonarqube.apps.itz-y6rg5f.infra01-lb.wdc07.techzone.ibm.com/api/projects/create?project=$PROJECT_KEY&name=Order%20Service%20${USER}"; echo ""; echo "TOKEN=$TOKEN"; echo "PROJECT_KEY=$PROJECT_KEY"; echo "Project created in SonarQube!"
   ```

1. Bob will run this command to:
   - Generates a unique SonarQube authentication token
   - Creates a unique project key with timestamp (e.g., `order-service-testuser1-1778261688`)
   - Creates the project in SonarQube via API
   - Displays the token and project key for the next step

1. Bob will return a token and project key. **Save both the TOKEN and PROJECT_KEY** - you'll need them for the next step.

1. Next, prompt to Bob generate an analysis report:

   ```
   Scan order-service with the newly created SonarQube project and generate a comprehensive analysis report
   ```

1. Bob will perform a comprehensive 5-step analysis process, which should include:

   - Bob will runs a scan using maven, which may fail the first time due to the injected vulnerabilities, but Bob will update its command and run something similar to: `cd order-service && mvn clean compile sonar:sonar -Dsonar.projectKey=order-service-testuser1-1778261688 -Dsonar.projectName="Order Service testuser1" -Dsonar.host.url=https://sonarqube-sonarqube.apps.itz-y6rg5f.infra01-lb.wdc07.techzone.ibm.com -Dsonar.token=squ_dc8b45225f4ebdcd40cc73a4a19d2b3b14f76755 -DskipTests`. Bob will automatically create the correct SonaryQube parameters based on the **PROJECT_KEY** and **TOKEN** from the prior step.
   - Bob will fetch Issues from SonarQube API, using a command similar to: `curl -s -u txcdemo:{REPLACE_SONARQUBE_PASSWORD} "https://sonarqube.../api/issues/search?componentKeys=order-service&resolved=false&ps=500"`
   - Bob will fetch security hotspots and vulnerability rating, using a command similar to: `curl -s -u txcdemo:{REPLACE_SONARQUBE_PASSWORD} "https://sonarqube.../api/hotspots/search?projectKey=order-service&ps=500"`
   - Bob will fetch project metrics (i.e. quality gate metrics and rating) as well as bugs, vulnerabilities, code smells, coverage, technical debt, using a command similar to: `curl -s -u txcdemo:{REPLACE_SONARQUBE_PASSWORD} "https://sonarqube.../api/measures/component?component=order-service&metricKeys=..."`
   - Finally, Bob will generate a comprehensive analysis report that includes: Executive summary with quality gate status, detailed breakdown of all issues, Security hotspots analysis with remediation code, Bugs and code smells with fixes, Comparison with manual security review, Remediation priority matrix, CI/CD integration recommendations, Quality gate configuration

   **Note:** Bob goes beyond just running the scan - it automatically fetches detailed data via API and creates a comprehensive markdown report with remediation guidance. This provides much more value than viewing results in the SonarQube UI!

1. Open and review the generated `SONARQUBE_ANALYSIS_REPORT.md`. Some of the key findings you would see:
   - ✅ **100% Correlation** - All 6 vulnerabilities identified by Bob were confirmed by SonarQube:
      - Hardcoded credentials ✓
      - Insecure logging ✓
      - Weak cryptography (MD5) ✓
      - Stack trace exposure ✓
      - Weak random generation ✓
      - Information exposure ✓
   - **Additional SonarQube Findings:**
      - Unused private fields
      - Unused method parameters
      - Generic wildcard usage
      - Math.abs edge case


## Part 4: Add SonarQube & Grype Security Analysis Stage

> **Note:** Before adding the security stage, ensure you have completed Part 3 (SonarQube token and project).

In this section, we will add SonarQube to the Jenkins pipeline. As we have seen, we can use Bob to create the Jenkinsfile stage. You can leverage Bob's capabilities by crafting a **comprehensive, specification-style prompts** that describe exactly what you need. Instead of asking "Add security scanning," we will provide a complete technical specification with:
- ✅ Exact commands and tool versions
- ✅ Container contexts and file paths
- ✅ Data structure requirements (serializable types only)
- ✅ Error handling and retry logic
- ✅ Environment constraints (no external dependencies, rate limits)
- ✅ Success criteria and logging requirements

1. Switch to **Jenkins Pipeline Integration** mode before proceeding with this step.

1. Prompt Bob with the following:

   ```
   Add a comprehensive Security Analysis stage to the Jenkinsfile that performs multi-layered security scanning. The stage should be placed after the existing stages and include:

   REQUIREMENTS:

   1. Environment Configuration:
      - Load SonarQube token from Jenkins credentials: credentials('sonarqube-token')
      - Set SONAR_HOST_URL to: https://sonarqube-sonarqube.apps.itz-y6rg5f.infra01-lb.wdc07.techzone.ibm.com
      - Set PROJECT_KEY dynamically using: "order-service-${env.USER}-${env.BUILD_NUMBER}"

   2. Phase 1 - SonarQube Security Scanning (MANDATORY):
      - Run Maven SonarQube scan from order-service directory
      - Use test-compile goal before sonar:sonar
      - **CRITICAL: Implement two-phase wait strategy:**
      
      **Phase 1a - Wait for Task Completion (12 retries, 10 seconds between):**
      - Check task status via: curl "${SONAR_HOST_URL}/api/ce/component?component=${PROJECT_KEY}"
      - Parse status using: grep -o '"status":"[^"]*"' | head -1 | cut -d'"' -f4
      - Continue only when status == 'SUCCESS'
      - Break if status == 'FAILED'
      - Use try-catch for error handling
      
      **Phase 1b - Fetch Metrics (6 retries, 5 seconds between):**
      - Only after task completion, fetch metrics via: curl "${SONAR_HOST_URL}/api/measures/component?component=${PROJECT_KEY}&metricKeys=bugs,vulnerabilities,code_smells,security_hotspots,security_rating,reliability_rating,security_review_rating"
      - Parse JSON response using Groovy JsonSlurper (no jq dependency)
      - Store metrics as String primitives in securityResults.sonarqube map
      - Check if sonarData.component?.measures exists and has size > 0
      - Set sonarData = null after parsing to avoid serialization issues
      
      - Store results in serializable Map structure with String values
      - Display metrics in console output

   3. Phase 2 - Grype Vulnerability Scanning:
      - Install Grype on-demand in the build-tools container using: curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /usr/local/bin
      - Run scan from order-service directory: grype dir:. --scope all-layers -o json > grype-report.json
      - Parse JSON to count Critical, High, and Medium severity vulnerabilities
      - Use grep and wc -l for counting (no jq dependency)
      - Store counts as Integer primitives in securityResults.trivy map
      - Wrap in catchError to prevent pipeline failure

   4. Phase 3 - CVE Analysis with Bob (conditional):
      - Only run if total vulnerabilities > 0
      - Use Bob in 'code' mode (not custom mode)
      - Provide detailed prompt including SonarQube and Grype results
      - Request: risk assessment, top 3 vulnerabilities, remediation steps, deployment recommendation
      - Save analysis to bob-cve-analysis.txt
      - Extract deployment recommendation (BLOCK/WARN/PROCEED)

   5. Phase 4 - Risk Level Calculation:
      - Parse SonarQube String values to Integer safely using Integer.parseInt()
      - Calculate risk score: vulnerabilities*10 + hotspots*5 + bugs*2 + critical_cves*20 + high_cves*10
      - Determine risk level: CRITICAL (≥50), HIGH (≥30), MEDIUM (≥10), LOW (<10)
      - Store as serializable primitives (String/Integer only)

   6. Phase 5 - Deployment Decision:
      - CRITICAL → BLOCK deployment, set currentBuild.result = 'UNSTABLE'
      - HIGH → WARN with manual review recommendation
      - MEDIUM/LOW → PROCEED with approval

   7. Phase 6 - Consolidated Report Generation:
      - Create markdown report: security-analysis-report.md
      - Include: executive summary, SonarQube results table, Grype results, CVE analysis, deployment recommendation
      - Archive artifacts: security-analysis-report.md, bob-cve-analysis.txt, order-service/grype-report.json

   IMPORTANT CONSTRAINTS:
   - Use only serializable data types (String, Integer, Boolean) in securityResults Map
   - No jq or external JSON parsers - use Groovy JsonSlurper for SonarQube, grep/wc for Grype
   - Install Grype in build-tools container (no separate security-tools container due to Docker Hub rate limits)
   - Use 15-minute timeout for the entire stage
   - Wrap all phases in catchError blocks to ensure pipeline completion
   - Use container('build-tools') for Grype installation and scanning
   - Use container('bob') for CVE analysis
   - **CRITICAL: Must implement two-phase SonarQube wait (task status check THEN metrics fetch)**
   - Store SonarQube metrics as Strings, parse to Integer only when calculating risk score
   - Clear JSON objects after parsing (set to null) to avoid serialization issues

   The stage should provide clear console output with emoji indicators and progress messages for each phase.
   ```

1. Bob generates a production-ready `Security Analysis` stage (for details on the structure see - ). Some of the key benefits of this approach:

   - ✅ **Mandatory SonarQube Integration** - Ensures consistent security scanning across all builds
   - ✅ **On-Demand Tool Installation** - Installs Grype dynamically to avoid Docker Hub rate limits
   - ✅ **Intelligent CVE Analysis** - Uses Bob's AI for risk assessment and remediation guidance
   - ✅ **Conditional Analysis** - Only runs detailed CVE analysis when vulnerabilities detected
   - ✅ **Comprehensive Risk Assessment** - Considers ALL security reports together
   - ✅ **No External Dependencies** - Uses Groovy JsonSlurper and grep/wc (no jq required)
   - ✅ **Clear Deployment Decisions** - Automated risk-based gates with BLOCK/WARN/PROCEED logic

   > **Important:** You will add the SonarQube token to Jenkins credentials in a later step. The Jenkinsfile you create in this step will reference the credential ID `sonarqube-token`, which you'll configure before running the build.

1. While this lab focuses on SonarQube as the mandatory security scanning tool, Bob's Software Security Reviewer mode (covered in Part 2) can be integrated into your pipeline for more comprehensive security analysis. However, this would require additional customization to fit your specific needs and workflow. The Software Security Reviewer mode is best used for:
   - Pre-deployment security audits
   - Compliance assessments (PCI DSS, OWASP)
   - Threat modeling and attack scenario analysis
   - Custom security requirements beyond standard code scanning

1. [OPTIONAL] If you wish to integrate the Software Security Reviewer mode into your pipeline, you can customize the security stage to include Bob-based analysis alongside SonarQube scanning.


## Part 5: Push and Watch

1. Lets commit and push the updates. Prompt Bob or run the following commands in the terminal:

   ```
   git add Jenkinsfile .bob/ labs/lab3/ order-service/src/main/java/com/example/orders/service/OrderService.java order-service/pom.xml
   git commit -m "Lab 3 — Security Analysis stage with vulnerability scanning and injected vulnerabilities for testing"
   git push
   ```

1. This will: 
   - Adds the updated Jenkinsfile with the Security Analysis stage
   - Adds any custom Bob modes from .bob/ directory
   - Adds lab3 files (scripts, documentation)
   - **Adds OrderService.java with injected vulnerabilities** for security testing
   - **Adds pom.xml with vulnerable Log4j dependency** for dependency scanning
   - Commits all changes with descriptive message
   - Pushes to trigger Jenkins build with vulnerabilities present

   > **Note:** We're intentionally committing the files with vulnerabilities so the Jenkins pipeline can detect them during the security scan. We'll remove these vulnerabilities and push clean code later in the lab.

1. Before running the build, you need to add your SonarQube token to Jenkins credentials so the pipeline can authenticate with SonarQube.

1. **Navigate to Jenkins Credentials:**
   - Go to Jenkins → **Manage Jenkins** → **Credentials**
   - Click **Add Credentials**

1. **Create the credential:**
   - **Kind:** Secret text
   - **Scope:** Global
   - **Secret:** Paste your SonarQube token from Step 3.1 (e.g., `squ_dc8b45225f4ebdcd40cc73a4a19d2b3b14f76755`)
   - **ID:** `sonarqube-token`
   - **Description:** SonarQube authentication token
   - Click **Create**

1. **Run the Build:**
   - Navigate to your pipeline in Jenkins
   - Click **Build Now**
   - Watch the console output as the pipeline executes

1. You should see the following:
   - `Checkout` and `PR Review` stages turn green
   - `Security Scan` stage runs with multiple security checks:
      - Secret scanning detects hardcoded credentials
      - SonarQube analysis identifies security hotspots
      - Dependency vulnerability scan runs
      - Code pattern checks flag insecure practices
      - Configuration security checks validate K8s/Docker configs
      - Risk assessment calculates overall security posture
   - Security reports are generated and archived as build artifacts:
      - `Security_Analysis_Report.md` - Bob's comprehensive security analysis
      - `SonarQube_Analysis_Report.md` - SonarQube findings
      - `CVE_ANALYSIS_REPORT.md` - CVE analysis (if vulnerabilities detected)
      - `SECURITY_REPORT.txt` - Pipeline security summary
   - Pipeline may turn **UNSTABLE (yellow)** or **FAILURE (red)** depending on severity of findings
   - Build page lists all security reports under **Build Artifacts**

1. Open the archived artifacts from the build page to review:
   - Bob's detailed security analysis with remediation guidance
   - SonarQube's quality gate status and security hotspots
   - CVE analysis with risk assessment and deployment recommendations

**Note:** In an ideal production environment, you would generate separate reports for each tool (SonarQube_Analysis_Report.md, CVE_ANALYSIS_REPORT.md, SECURITY_REPORT.txt). However, for demo purposes, this lab consolidates all findings into a single `security-analysis-report.md` that includes:
   - Executive summary with risk level and deployment decision
   - SonarQube analysis results (bugs, vulnerabilities, security hotspots, code smells)
   - Grype vulnerability scan results (Critical, High, Medium severity counts)
   - Risk assessment calculation and thresholds
   - Deployment recommendation (BLOCK/WARN/PROCEED)
   - Next steps based on risk level

**Note:** The security stage is designed to block deployment if CRITICAL issues are detected. Review the console output and security reports to understand what needs to be fixed before the next deployment.

## Part 6: Restore Original Code

After the Jenkinsfile changes have been pushed and the build has run, we need to remove the vulnerabilities from OrderService.java and pom.xml that were committed to the repository.

1. Switch to **Agent mode** and prompt Bob with:

   ```
   Revert OrderService.java and pom.xml to their original state before vulnerability injection by checking out the version from before the vulnerabilities were added, then commit and push these changes
   ```
1. Bob will:
   - Identifies the commit before vulnerabilities were injected
   - Restores the clean versions of the files:
      - `git checkout <commit-hash> -- order-service/src/main/java/com/example/orders/service/OrderService.java`
      - `git checkout <commit-hash> -- order-service/pom.xml`
   - Commits the restored files:
      - `git add order-service/src/main/java/com/example/orders/service/OrderService.java order-service/pom.xml`
      - `git commit -m "Restore OrderService.java and pom.xml to clean state - remove injected vulnerabilities"`
   - Pushes the changes:
      - `git push`

1. If you prefer to revert the entire vulnerability injection commit:
   ```bash
   git log --oneline  # Find the commit hash of vulnerability injection
   git revert <commit-hash>  # Creates a new commit that undoes the changes
   git push
   ```

1. In Jenkins, click **Build Now** on your pipeline and watch the console.

1. This time, the pipeline should complete.

---

## Lab Summary

### What You Accomplished

✅ **Injected Vulnerabilities:**
- 6 types of security vulnerabilities in OrderService.java
- Realistic security issues (hardcoded credentials, weak crypto, etc.)

✅ **Manual Security Analysis:**
- Generated comprehensive Security Analysis Report
- Identified all vulnerabilities with remediation guidance
- Mapped to CWE and PCI DSS requirements

✅ **Automated Security Scanning:**
- Created SonarQube token
- Ran automated code quality and security scan
- Generated SonarQube Analysis Report
- Verified 100% correlation with manual findings

✅ **Pipeline Enhancement:**
- Added comprehensive Security Analysis stage to Jenkinsfile
- Implemented 6-layer security scanning (Secret Scanning, SonarQube, Dependency Scan, Code Patterns, Configuration Checks, Risk Assessment)
- Created risk-based deployment gates (CRITICAL/HIGH/MEDIUM/LOW)
- Added automated security metrics tracking and comprehensive report generation
- Set deployment gates that BLOCK deployment on CRITICAL issues

✅ **CVE Analysis Framework:**
- Created reusable CVE analysis prompt
- Standardized vulnerability assessment process
- Integrated compliance requirements (PCI DSS)

### Key Takeaways

1. **Defense in Depth:** Multiple security scanning layers catch different types of issues
2. **Automation + Manual:** Automated tools (SonarQube) validate manual analysis (Bob)
3. **Actionable Intelligence:** Reports provide specific remediation steps, not just findings
4. **Compliance Focus:** Security analysis tied to regulatory requirements (PCI DSS)
5. **Pipeline Integration:** Security gates prevent vulnerable code from reaching production

---

## Troubleshooting

- **Tests fail after vulnerability injection** - This is expected behavior since the injected vulnerabilities break the validation logic. Use `-DskipTests` to continue: `mvn clean compile -DskipTests`
- **Bob can't find files** - Make sure you are in the correct directory. The command `pwd` should return this repo
- **Pipeline security stage fails** - Verify that SonarQube is accessible, your SonarQube Token is set (`echo $SONAR_TOKEN`), and that Maven can compile (`cd order-service && mvn compile`)
- **Bob completes the analysis but doesn't create a report file** - Use a follow-up prompt to ask Bob to create the missing reports (`Create the SECURITY_ANALYSIS_REPORT.md file with all the findings you just analyzed`)

---

## Additional Information and Resources

### Resources
- [SonarQube Security Rules](https://rules.sonarsource.com/java/type/Vulnerability)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [PCI DSS Requirements](https://www.pcisecuritystandards.org/)
- [CWE Top 25](https://cwe.mitre.org/top25/)
- [Trivy](https://github.com/aquasecurity/trivy) - Container vulnerability scanner
- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/) - Dependency vulnerabilities
- [Checkmarx](https://checkmarx.com/) - SAST scanning

### Information and Tips

- There is a cleanup script in the repo that restores the changes made to `OrderService.java` and removes the backup. You can run the script by asking bob: `Run script restore_vulnerabilities.sh`
- If you get stuck, you can restore changes that have not been committed yet using Git. For example, to revert a change made to the Jenkinsfile, use the command: `git checkout Jenkinsfile`
- Details for the `Security Analysis` stage you are adding can be found in the [security-stage-details.md](security-stage-details.md)