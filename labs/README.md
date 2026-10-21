# Labs Overview

Hands-on labs to learn how DevOps and automation lifecycles can be improved with IBM Bob AI. From development-time analysis and insights to including AI automations in your CI/CD pipeline.

---

## Table of Contents

- [Getting Started](#-getting-started)
- [Workshop Labs](#-workshop-labs)
- [After Completing the Labs](#-after-completing-the-labs)
- [Additional Resources](#-additional-resources)

---

## 🚀 Getting Started

### Prerequisites

Before starting any labs, complete the setup: follow the [Setup Instructions](lab0/SETUP.md) to configure Jenkins, create your branch, and set up your pipeline.

### Workshop Structure

1. **Workshop Labs** (Labs 1-5) - CI/CD pipeline automation with Bob and Jenkins

---

## 🔧 Workshop Labs

**CI/CD Pipeline Automation with Jenkins and Bob**

These labs build a complete AI-assisted Jenkins pipeline, adding one stage per lab. Each lab introduces a new Bob custom mode and pipeline integration pattern.

### Lab 0: Setup

**Time**: 10 minutes | **File**: [lab0/SETUP.md](lab0/SETUP.md)

**What You'll Do:**

- Create a GitHub PAT (used to `git push` to your fork)
- Fork the repository, cache your Git credentials, and create your working branch
- Set up your Jenkins pipeline
- Run initial verification build

---

### Lab 1: PR/Git Diff Review with Bob

**Time**: 15 minutes | **File**: [lab1/PR_REVIEW.md](lab1/PR_REVIEW.md)

**What You'll Do:**

- Use the `askBob` helper function for Jenkins
- Use Bob in Advanced mode to analyze git diffs
- Create the `pipeline-git-diff-overview` custom mode
- Use Bob to conversationally build the PR Review stage in your Jenkinsfile

**📚 What You'll Learn:**

- Writing reusable Jenkins helper functions
- Creating read-only pipeline modes
- Structuring Jenkinsfile stages for AI integration
- Git diff summarization in CI context

**Use Case**: Automated PR review and change analysis in every build

---

### Lab 2: Unit Testing with Bob

**Time**: 15 minutes | **File**: [lab2/UNIT_TESTING.md](lab2/UNIT_TESTING.md)

**What You'll Do:**

- Create `java-unit-test-mode` for writing tests in the IDE
- Use Bob to generate and run unit tests locally
- Create `pipeline-test-failure-analyzer` mode
- Add Unit Tests stage to Jenkinsfile
- Use Bob to write the prompt for pipeline test failure analysis

**📚 What You'll Learn:**

- AI-assisted unit test generation with JUnit 5 and Mockito
- Running and interpreting test results locally
- Diagnosing test failures using Bob as a triage tool
- Publishing JUnit results in Jenkins

**Use Case**: Automated test generation and intelligent test failure diagnosis

---

### Lab 3: Security Vulnerability Analysis

**Time**: 25 minutes | **File**: [lab3/SECURITY_ANALYSIS.md](lab3/SECURITY_ANALYSIS.md)

**What You'll Do:**

- Inject security vulnerabilities and observe Bob Findings in real-time
- Use Software Security Reviewer mode for comprehensive analysis
- Run SonarQube security scans
- Run Grype container vulnerability scans
- Add Security Analysis stage with SonarQube and Grype integration

**📚 What You'll Learn:**

- IDE-integrated security findings with Bob
- Comprehensive security analysis (code, infrastructure, compliance)
- SonarQube integration and CVE triage
- Container vulnerability scanning with Grype
- Understanding shift-left security principles

**Use Case**: Shift-left security with automated vulnerability detection and AI-assisted triage

---

### Lab 4: Linting and Compliance

**Time**: 15 minutes | **File**: [lab4/LINTING.md](lab4/LINTING.md)

**What You'll Do:**

- Use Checkstyle for Java code quality linting
- Run Hadolint (Dockerfile), Checkov (IaC), and KubeLinter (K8s)
- Use Bob to create `pipeline-lint-analyzer` mode
- Use Bob to add Linting stages that aggregate multi-tool output

**📚 What You'll Learn:**

- Multi-layer linting (code, container, infrastructure, Kubernetes)
- Interpreting and prioritizing linter findings with AI
- Identifying overlapping issues across tools
- Producing consolidated lint reports
- Feeding CI/CD reports to Bob CLI for analysis

**Use Case**: Enforce policy standards automatically with AI-powered analysis and prioritization

---

### [Optional] Lab 5: Jenkins Pipeline Insights with MCP

**Time**: 15 minutes | **File**: [lab5/JENKINS_MCP.md](lab5/JENKINS_MCP.md)

**What You'll Do:**

- Configure Jenkins MCP server in Bob
- Query pipeline status and build history from Bob
- Analyze build failures without leaving the IDE
- Generate automated insights on pipeline health

**📚 What You'll Learn:**

- Jenkins MCP server setup and configuration
- Querying Jenkins data through Bob
- Automated build failure analysis
- Pipeline monitoring and insights generation
- Correlating build failures with code changes

**Use Case**: Monitor and analyze Jenkins pipelines entirely through Bob, eliminating context switching

**Prerequisites**: Labs 1-4 complete (to have pipeline history)

---

## 🚀 Working with Bob

**Tips:**

- ✅ Copy prompts exactly as shown in labs
- ✅ Include `@` file references when specified — context matters
- ✅ Read Bob's response before running any generated code
- ✅ Ask follow-up questions if something is unclear
- ✅ Use Bob's custom modes for specialized tasks
- ✅ Check troubleshooting sections if you get stuck
- ✅ **Remember: You can always ask Bob for help**

---

## 🎯 After Completing the Labs

### What's Next?

**Extend Your Pipelines:** Feel free to explore making your own additions / extensions to what you learned in the labs. Some ideas to explore:

- Add pre-commit Bob mode for diff analysis before every push
- Integrate GitHub MCP server for automated PR summaries
- Create auto-recovery and self-healing runbooks with Bob
- Build custom modes for your team's specific workflows

**Apply to Your Projects:** Apply what you've learned to your own pipelines, and see how Bob can help you.

- Identify manual pipeline stages that could be automated
- Start with the `askBob` pattern from Lab 1 — works with any CI system
- Use Bob's custom modes to encode team-specific knowledge
- Integrate Bob Findings for continuous code quality monitoring

---

## 🎉 Congratulations!

After completing the workshop labs, you'll know how to:

- ✅ Create custom modes for your workflows
- ✅ Integrate Bob into Jenkins CI/CD pipelines as an AI agent
- ✅ Generate, run, and triage unit tests with Bob
- ✅ Surface and explain security vulnerabilities using Bob Findings
- ✅ Enforce and understand linting policies across multiple tools
- ✅ Monitor Jenkins pipelines through Bob with MCP

---

## 📚 Additional Resources

- **Bob Documentation**: [https://bob.ibm.com/docs](https://bob.ibm.com/docs)
- **Custom Modes Guide**: [https://bob.ibm.com/docs/ide/configuration/custom-modes](https://bob.ibm.com/docs/ide/configuration/custom-modes)
- **MCP Protocol**: [https://modelcontextprotocol.io](https://modelcontextprotocol.io)

---

**Ready to start?** Start with the Setup lab above and dive in! 🚀