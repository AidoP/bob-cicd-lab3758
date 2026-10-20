# Labs Overview

Hands-on labs to learn how DevOps and automation lifecycles can be improved with IBM Bob AI. From development-time analysis and insights to including AI automations in your CI/CD pipeline.

---

## Table of Contents

- [Getting Started](#-getting-started)
- [Workshop Labs](#-workshop-labs)
- [Handoff Labs: Advanced Topics](#-handoff-labs-advanced-topics)
- [Quick Start Guide](#-quick-start)
- [After Completing the Labs](#-after-completing-the-labs)
- [Additional Resources](#-additional-resources)

---

## 🚀 Getting Started

### Prerequisites

Before starting any labs, complete the setup: follow the [Setup Instructions](00_SETUP.md) to configure Jenkins, create your branch, and set up your pipeline.

### Workshop Structure

1. **Workshop Labs** (Labs 1-4 and 6) - CI/CD pipeline automation with Bob and Jenkins
2. **Handoff Labs** (6 labs) - Optional advanced topics and custom integrations

---

## 🔧 Workshop Labs

**CI/CD Pipeline Automation with Jenkins and Bob**

These labs build a complete AI-assisted Jenkins pipeline, adding one stage per lab. Each lab introduces a new Bob custom mode and pipeline integration pattern.

### Lab 0: Setup

**Time**: 10 minutes | **File**: [00_SETUP.md](00_SETUP.md)

**What You'll Do:**

- Create your GitHub PAT (if using private repo)
- Fork the repository and create your working branch
- Set up your Jenkins pipeline
- Run initial verification build

---

### Lab 1: PR/Git Diff Review with Bob

**Time**: 20 minutes | **File**: [lab1/LAB1_PR_REVIEW.md](lab1/LAB1_PR_REVIEW.md)

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

**Time**: 20 minutes | **File**: [lab2/LAB2_UNIT_TESTING.md](lab2/LAB2_UNIT_TESTING.md)

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

**Time**: 30 minutes | **File**: [lab3/LAB3_SECURITY_ANALYSIS.md](lab3/LAB3_SECURITY_ANALYSIS.md)

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

**Time**: 20 minutes | **File**: [lab4/LAB4_LINTING.md](lab4/LAB4_LINTING.md)

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

### Lab 6: Jenkins Pipeline Insights with MCP

**Time**: 15 minutes | **File**: [lab6/LAB6_JENKINS_MCP.md](lab6/LAB6_JENKINS_MCP.md)

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

## 🎓 Handoff Labs: Advanced Topics

These optional labs demonstrate advanced Bob capabilities and real-world integration patterns.

### 1. Bob Mode Builder

**Time**: 30 min - 2+ hours | **File**: [handoff-labs/bob-mode-labs/README.md](handoff-labs/bob-mode-labs/README.md)

Create custom Bob modes for your specific workflows with templates and examples.

**Topics**: Mode architecture, workflow modes, 5 example modes, templates

**Prerequisites**: Basic Bob familiarity

---

### 2. Example MCP Servers

**Time**: 30 min - 3+ hours | **File**: [handoff-labs/bob-mcp-labs/README.md](handoff-labs/bob-mcp-labs/README.md)

Five production-ready MCP server examples using FastMCP framework.

**Topics**: Calculator, file operations, database operations, API integration

**Prerequisites**: Python 3.8+, UV package manager

---

### 3. OpenShift Operations with Bob

**Time**: 45-60 minutes | **File**: [handoff-labs/openshift-with-bob/README.md](handoff-labs/openshift-with-bob/README.md)

Manage Red Hat OpenShift Container Platform using Bob's AI-native approach.

**Topics**: MCP-based automation, custom mode workflows, deployment, monitoring, troubleshooting

**Prerequisites**: OpenShift CLI (`oc`), access to OpenShift cluster, Python 3.8+, UV

---

### 4. Python to JavaScript Translation

**Time**: 20 minutes | **File**: [handoff-labs/python-to-javascript/README.md](handoff-labs/python-to-javascript/README.md)

Translate code from one programming language to another while maintaining functionality.

**Topics**: Language analysis, translation planning, feature mapping, verification

**Prerequisites**: Python 3.8+, Node.js 14+, UV package manager

---

### 5. Simple App Development

**Time**: 30-45 minutes | **File**: [handoff-labs/simple-app-development/README.md](handoff-labs/simple-app-development/README.md)

Build a complete full-stack application from scratch with Bob.

**Topics**: Full-stack development, Flask REST API, SQLite, HTML/CSS/JavaScript, CRUD operations

**Prerequisites**: Python 3.8+, Node.js 14+

---

### 6. Java Application Modernization

**Time**: 45-60 minutes | **File**: [handoff-labs/simple-java-modernization/README.md](handoff-labs/simple-java-modernization/README.md)

Modernize legacy Java applications from Java 8 to Java 17/21.

**Topics**: Records, sealed classes, pattern matching, switch expressions, API updates, virtual threads

**Prerequisites**: Java 17 or 21, Maven or Gradle

---

## 🚀 Quick Start

### Choose Your Path

1. **Start here** → Follow the [Workshop Labs](#-workshop-labs) (complete Setup first)
2. **Want to go further?** → Explore the [Handoff Labs](#-handoff-labs-advanced-topics)

### Working with Bob

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

**Explore Handoff Labs:** There are a few additional labs that build on top of the concepts learned in this workshop

- Advanced topics building on workshop concepts
- Real-world integration patterns
- Custom mode development
- MCP server examples

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
- **Bob Differentiators**: [resources/bob-differentiators.md](../resources/bob-differentiators.md)
- **Custom Modes Guide**: [https://bob.ibm.com/docs/ide/configuration/custom-modes](https://bob.ibm.com/docs/ide/configuration/custom-modes)
- **MCP Protocol**: [https://modelcontextprotocol.io](https://modelcontextprotocol.io)

---

**Ready to start?** Start with the Setup lab above and dive in! 🚀