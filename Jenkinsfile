// ═══════════════════════════════════════════════════════════════════
// Base Pipeline (Jenkinsfile)
//
// Starting point for the workshop. Provisions a Kubernetes
// agent pod with three containers — build-tools (Maven + JDK17),
// oc-tools (OpenShift CLI), and bob (IBM Bob CLI) — sharing a
// workspace-volume emptyDir mounted at /workspace on all three,
// with HOME set to /workspace so Bob can read .bob/custom_modes.yaml
// from the checkout.
//
// Before running: if your OpenShift project is NOT named `jenkins`,
// update the `bob` container image URL below.
// ═══════════════════════════════════════════════════════════════════
pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: build-tools
    image: maven:3.9-eclipse-temurin-17
    command: ['sleep', 'infinity']
    workingDir: /workspace
    volumeMounts:
    - name: workspace-volume
      mountPath: /workspace
    env:
    - name: HOME
      value: /workspace
  - name: oc-tools
    image: quay.io/openshift/origin-cli:latest
    command: ['sleep', 'infinity']
    workingDir: /workspace
    volumeMounts:
    - name: workspace-volume
      mountPath: /workspace
    env:
    - name: HOME
      value: /workspace
  - name: lint-tools
    image: image-registry.openshift-image-registry.svc:5000/jenkins/lint-tools:latest
    command: ['sleep', 'infinity']
    workingDir: /workspace
    volumeMounts:
    - name: workspace-volume
      mountPath: /workspace
    env:
    - name: HOME
      value: /workspace
  - name: bob
    image: image-registry.openshift-image-registry.svc:5000/jenkins/bob-cli:latest
    command: ['sleep', 'infinity']
    workingDir: /workspace
    volumeMounts:
    - name: workspace-volume
      mountPath: /workspace
    env:
    - name: BOBSHELL_API_KEY
      valueFrom:
        secretKeyRef:
          name: bob-cli-credentials
          key: BOBSHELL_API_KEY
    - name: BOB_ACCEPT_LICENSE
      value: "true"
    - name: HOME
      value: /workspace
  volumes:
  - name: workspace-volume
    emptyDir: {}
"""
            defaultContainer 'build-tools'
        }
    }

    stages {
        stage('Checkout') {
            steps {
                echo "=== Checking out repository ==="
                checkout scm

                echo "=== Verifying workspace contents ==="
                sh '''
                    echo "Current directory: $(pwd)"
                    echo "Workspace contents:"
                    ls -la
                '''
            }
        }

        // ── Lab 1: PR / Git Diff Review ──────────────────────────
        stage('PR Review') {
            options {
                timeout(time: 5, unit: 'MINUTES')
            }
            steps {
                script {
                    echo '════════════════════════════════════════════════════════'
                    echo '  Bob PR Review Analysis'
                    echo "  Started: ${new Date()}"
                    echo '════════════════════════════════════════════════════════'

                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                        sh '''
                            git config --global --add safe.directory "$WORKSPACE"
                            git diff origin/main...HEAD > git-diff.txt || : > git-diff.txt
                        '''

                        def prompt = """
Read git-diff.txt and produce a senior developer PR overview.

Focus on:
- What changed (1-2 sentences per notable change)
- Risk level (high/medium/low) for each change
- Watch for: null-safety, concurrency, behavior changes, security surface, performance hot paths, missing tests

Keep it brief - if nothing notable changed, say so in one sentence.
"""
                        def analysis = askBob(prompt, 'pipeline-git-diff-overview')

                        echo ''
                        echo analysis
                        echo ''

                        writeFile file: 'bob-pr-review.md', text: analysis
                    }

                    echo "  Completed: ${new Date()}"
                    echo "  Full report: bob-pr-review.md (archived)"
                    echo '════════════════════════════════════════════════════════'
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'bob-pr-review.md',
                                   allowEmptyArchive: true,
                                   fingerprint: true
                }
            }
        }

        // ── Lab 2: Unit Testing ──────────────────────────────────
        stage('Unit Tests') {
            options {
                timeout(time: 10, unit: 'MINUTES')
            }
            steps {
                script {
                    echo '════════════════════════════════════════════════════════'
                    echo '  Running Unit Tests'
                    echo "  Started: ${new Date()}"
                    echo '════════════════════════════════════════════════════════'

                    def testResult = sh(
                        script: 'mvn -f order-service/pom.xml test',
                        returnStatus: true
                    )

                    junit testResults: 'order-service/target/surefire-reports/*.xml',
                          allowEmptyResults: true

                    if (testResult != 0) {
                        echo ''
                        echo '  Tests failed - analyzing with Bob...'
                        echo ''

                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            def prompt = """
Analyze the JUnit test failures in order-service/target/surefire-reports/.

For each failed test:
- One sentence on what failed and where
- Likely root cause (assertion mismatch, NPE, mock misconfiguration, timeout, wrong exception type)
- Suggested fix with a short code snippet
- Group related failures if they share a root cause

Read the test source files under order-service/src/test/ and production code under order-service/src/main/ as needed.

Output in plain text with sections: Summary, Failures, Suggested Fixes.
"""
                            def analysis = askBob(prompt, 'pipeline-test-failure-analyzer')

                            echo ''
                            echo '════════════════════════════════════════════════════════'
                            echo '  Bob Test Failure Analysis'
                            echo '════════════════════════════════════════════════════════'
                            echo analysis
                            echo '════════════════════════════════════════════════════════'
                            echo ''

                            writeFile file: 'bob-test-analysis.md', text: analysis
                        }

                        currentBuild.result = 'UNSTABLE'
                    } else {
                        echo ''
                        echo '  All tests passed'
                    }

                    echo ''
                    echo "  Completed: ${new Date()}"
                    if (testResult != 0) {
                        echo "  Test analysis: bob-test-analysis.md (archived)"
                    }
                    echo '════════════════════════════════════════════════════════'
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'bob-test-analysis.md',
                                   allowEmptyArchive: true,
                                   fingerprint: true
                }
            }
        }

        // ── Lab 3: Security Analysis ─────────────────────────────
        stage('Security Analysis') {
            options {
                timeout(time: 15, unit: 'MINUTES')
            }
            environment {
                SONAR_TOKEN    = credentials('sonarqube-token')
                SONAR_HOST_URL = 'https://sonarqube-sonarqube.apps.itz-y6rg5f.infra01-lb.wdc07.techzone.ibm.com'
                PROJECT_KEY    = "order-service-user7"
            }
            steps {
                script {
                    echo '════════════════════════════════════════════════════════'
                    echo '  Multi-Layer Security Analysis'
                    echo "  Started: ${new Date()}"
                    echo '════════════════════════════════════════════════════════'

                    def securityResults = [sonarqube: [:], grype: [:], riskLevel: '', riskScore: 0, deploymentDecision: '']

                    // Phase 1: SonarQube
                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                        echo ''
                        echo 'Phase 1: SonarQube Security Scanning'
                        sh """
                            cd order-service
                            mvn clean test-compile sonar:sonar \
                                -Dsonar.projectKey=${PROJECT_KEY} \
                                -Dsonar.projectName="Order Service user7" \
                                -Dsonar.host.url=${SONAR_HOST_URL} \
                                -Dsonar.token=\$SONAR_TOKEN \
                                -DskipTests
                        """
                        echo 'SonarQube scan completed'

                        sleep(15)

                        def metricsJson = sh(
                            script: """
                                curl -s -u "\$SONAR_TOKEN:" \
                                    "${SONAR_HOST_URL}/api/measures/component?component=${PROJECT_KEY}&metricKeys=bugs,vulnerabilities,code_smells,security_hotspots,security_rating,reliability_rating"
                            """,
                            returnStdout: true
                        ).trim()

                        if (metricsJson?.contains('"component"')) {
                            def sonarData = null
                            try {
                                sonarData = new groovy.json.JsonSlurper().parseText(metricsJson)
                                sonarData?.component?.measures?.each { m ->
                                    securityResults.sonarqube[m.metric] = m.value ?: '0'
                                }
                                echo "Bugs: ${securityResults.sonarqube.bugs ?: '0'}"
                                echo "Vulnerabilities: ${securityResults.sonarqube.vulnerabilities ?: '0'}"
                                echo "Security Hotspots: ${securityResults.sonarqube.security_hotspots ?: '0'}"
                            } finally { sonarData = null }
                        }
                    }

                    // Phase 2: Grype
                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                        echo ''
                        echo 'Phase 2: Grype Vulnerability Scanning'
                        container('build-tools') {
                            sh 'curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /usr/local/bin'
                            sh 'cd order-service && grype dir:. --scope all-layers -o json > grype-report.json || true'

                            def critical = sh(script: 'grep -o \'"severity":"Critical"\' order-service/grype-report.json | wc -l || echo 0', returnStdout: true).trim().toInteger()
                            def high     = sh(script: 'grep -o \'"severity":"High"\' order-service/grype-report.json | wc -l || echo 0', returnStdout: true).trim().toInteger()
                            def medium   = sh(script: 'grep -o \'"severity":"Medium"\' order-service/grype-report.json | wc -l || echo 0', returnStdout: true).trim().toInteger()

                            securityResults.grype.critical = critical
                            securityResults.grype.high     = high
                            securityResults.grype.medium   = medium
                            securityResults.grype.total    = critical + high + medium

                            echo "Critical: ${critical}, High: ${high}, Medium: ${medium}"
                        }
                    }

                    // Phase 3: Bob CVE Analysis (conditional)
                    def totalVulns = (securityResults.grype.total ?: 0)
                    if (totalVulns > 0) {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            echo ''
                            echo 'Phase 3: CVE Analysis with Bob'
                            def cvePrompt = """
Analyze security findings from our multi-layer scan:

SonarQube: Bugs=${securityResults.sonarqube.bugs ?: '0'}, Vulnerabilities=${securityResults.sonarqube.vulnerabilities ?: '0'}, Hotspots=${securityResults.sonarqube.security_hotspots ?: '0'}
Grype: Critical=${securityResults.grype.critical}, High=${securityResults.grype.high}, Medium=${securityResults.grype.medium}

Provide:
1. Risk Assessment (1-2 sentences)
2. Top 3 Vulnerabilities to address
3. Remediation Steps for top issues
4. End with exactly: DEPLOYMENT RECOMMENDATION: [BLOCK|WARN|PROCEED]
"""
                            def cveAnalysis = askBob(cvePrompt, null)
                            echo cveAnalysis
                            writeFile file: 'bob-cve-analysis.txt', text: cveAnalysis

                            if      (cveAnalysis.contains('DEPLOYMENT RECOMMENDATION: BLOCK'))   { securityResults.deploymentDecision = 'BLOCK' }
                            else if (cveAnalysis.contains('DEPLOYMENT RECOMMENDATION: WARN'))    { securityResults.deploymentDecision = 'WARN' }
                            else                                                                   { securityResults.deploymentDecision = 'PROCEED' }
                        }
                    } else {
                        echo 'Phase 3: No vulnerabilities found, skipping CVE analysis'
                        securityResults.deploymentDecision = 'PROCEED'
                    }

                    // Phase 4: Risk calculation & deployment decision
                    def bugs    = Integer.parseInt(securityResults.sonarqube.bugs ?: '0')
                    def vulns   = Integer.parseInt(securityResults.sonarqube.vulnerabilities ?: '0')
                    def spots   = Integer.parseInt(securityResults.sonarqube.security_hotspots ?: '0')
                    def rScore  = (vulns * 10) + (spots * 5) + (bugs * 2) +
                                  ((securityResults.grype.critical ?: 0) * 20) +
                                  ((securityResults.grype.high ?: 0) * 10)

                    securityResults.riskScore = rScore
                    securityResults.riskLevel = rScore >= 50 ? 'CRITICAL' : rScore >= 30 ? 'HIGH' : rScore >= 10 ? 'MEDIUM' : 'LOW'

                    if (securityResults.riskLevel == 'CRITICAL') { currentBuild.result = 'UNSTABLE' }

                    def report = """# Security Analysis Report
Build: ${env.BUILD_NUMBER} | Date: ${new Date()} | Project: ${PROJECT_KEY}

## Risk: ${securityResults.riskLevel} (score: ${rScore}) | Decision: ${securityResults.deploymentDecision}

## SonarQube
Bugs: ${securityResults.sonarqube.bugs ?: '0'} | Vulnerabilities: ${securityResults.sonarqube.vulnerabilities ?: '0'} | Hotspots: ${securityResults.sonarqube.security_hotspots ?: '0'}
Dashboard: ${SONAR_HOST_URL}/dashboard?id=${PROJECT_KEY}

## Grype
Critical: ${securityResults.grype.critical ?: 0} | High: ${securityResults.grype.high ?: 0} | Medium: ${securityResults.grype.medium ?: 0} | Total: ${securityResults.grype.total ?: 0}

## CVE Analysis
${totalVulns > 0 ? 'See bob-cve-analysis.txt for details.' : 'No vulnerabilities detected.'}
"""
                    writeFile file: 'security-analysis-report.md', text: report
                    echo "Risk: ${securityResults.riskLevel} | Decision: ${securityResults.deploymentDecision}"
                    echo '════════════════════════════════════════════════════════'
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'security-analysis-report.md,bob-cve-analysis.txt,order-service/grype-report.json',
                                   allowEmptyArchive: true,
                                   fingerprint: true
                }
            }
        }

        // ── Lab 4: Linting ───────────────────────────────────────
        stage('Run Linters') {
            options {
                timeout(time: 10, unit: 'MINUTES')
            }
            steps {
                script {
                    echo '════════════════════════════════════════════════════════'
                    echo '  Running Linters'
                    echo "  Started: ${new Date()}"
                    echo '════════════════════════════════════════════════════════'

                    sh 'mkdir -p lint-results'

                    catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                        sh '''
                            mvn -f order-service/pom.xml checkstyle:check || true
                            cp order-service/target/checkstyle-result.xml lint-results/checkstyle.xml || true
                        '''

                        container('lint-tools') {
                            sh 'hadolint order-service/Dockerfile > lint-results/hadolint.txt 2>&1 || true'
                            sh 'checkov -d order-service/deploy-flawed/ --compact --quiet > lint-results/checkov.txt 2>&1 || true'
                            sh 'kube-linter lint order-service/deploy-flawed/ > lint-results/kubelinter.txt 2>&1 || true'
                        }
                    }

                    echo "  Completed: ${new Date()}"
                    echo '════════════════════════════════════════════════════════'
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'lint-results/*',
                                   allowEmptyArchive: true,
                                   fingerprint: true
                }
            }
        }

        stage('Bob Lint Analysis') {
            when {
                expression {
                    return fileExists('lint-results/checkstyle.xml') ||
                           fileExists('lint-results/hadolint.txt')   ||
                           fileExists('lint-results/checkov.txt')    ||
                           fileExists('lint-results/kubelinter.txt')
                }
            }
            options {
                timeout(time: 10, unit: 'MINUTES')
            }
            steps {
                script {
                    echo '════════════════════════════════════════════════════════'
                    echo '  Bob Lint Analysis'
                    echo "  Started: ${new Date()}"
                    echo '════════════════════════════════════════════════════════'

                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                        def prompt = """
Analyze the linting results and provide a consolidated report.

Read and analyze:
- lint-results/checkstyle.xml
- lint-results/hadolint.txt
- lint-results/checkov.txt
- lint-results/kubelinter.txt
- order-service/src/
- order-service/Dockerfile
- order-service/deploy-flawed/

Sections: Executive Summary, Highest Priority Findings, Findings by Tool, Cross-Tool Themes, Recommended Fix Order.
"""
                        def analysis = askBob(prompt, 'pipeline-lint-analyzer')
                        writeFile file: 'bob-lint-report.md', text: analysis
                        echo 'Full lint report saved: bob-lint-report.md'

                        def commentPrompt = """
Read bob-lint-report.md and create a concise PR comment summary (max 500 words).
Start with overall assessment, list top 3-5 critical findings with file locations, and provide a quick action list.
"""
                        def prComment = askBob(commentPrompt, 'pipeline-lint-analyzer')
                        writeFile file: 'bob-lint-pr-comment.md', text: prComment
                        echo 'PR comment saved: bob-lint-pr-comment.md'

                        echo ''
                        echo analysis
                    }

                    echo "  Completed: ${new Date()}"
                    echo '════════════════════════════════════════════════════════'
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'bob-lint-report.md,bob-lint-pr-comment.md',
                                   allowEmptyArchive: true,
                                   fingerprint: true
                }
            }
        }
    }

    post {
        always {
            echo "=== Pipeline Complete ==="
            echo "Result: ${currentBuild.result ?: 'SUCCESS'}"
        }
    }
}

// ── Helper: ask Bob, optionally with a specific custom mode ───────────────────
// Writes the prompt to a tempfile in the shared workspace and runs `bob` in
// the bob container.
// Returns the analysis as a string. Using a tempfile (instead of inlining the
// prompt on the command line) avoids shell-escaping issues when the prompt
// contains quotes, backticks, or newlines — common with diffs.
def askBob(String prompt, String mode = null) {
    container('bob') {
        def promptFile = ".bob-prompt-${System.currentTimeMillis()}.txt"
        writeFile file: promptFile, text: prompt

        def modeFlag = mode ? "--mode ${mode}" : ""
        def analysis = sh(
            script: """cat ${promptFile} | bob run ${modeFlag} --accept-license""",
            returnStdout: true
        ).trim()

        sh "rm -f ${promptFile}"
        return analysis
    }
}
