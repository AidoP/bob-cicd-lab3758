# Workshop Prerequisites

## Universal (everyone)

- [Bob IDE](https://bob.ibm.com/docs/ide/getting-started/install-and-setup)
- [Git](https://git-scm.com/downloads)
- A terminal
- A modern web browser

## Workshop Labs

- [Java 17](https://adoptium.net/temurin/releases/?version=17)
- [Maven](https://maven.apache.org/install.html)
- [`uv`](https://docs.astral.sh/uv/getting-started/installation/) — single install, provides both `uv` and `uvx` (used by the Jenkins MCP server Lab)

## Credentials (handed out by your instructor)

Jenkins URL, Jenkins username + password, repo URL (public; you'll fork it)

You'll create your own GitHub PAT during [Lab 0 setup](labs/00_SETUP.md) to push to your fork.

## Verify

```bash
git --version
uv --version
java --version
mvn --version
```
