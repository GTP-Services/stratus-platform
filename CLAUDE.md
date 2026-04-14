# Stratus Platform

Platform team utilities for GTP-Services.

## Available Skills

| Skill | What it does |
|-------|-------------|
| `/seal-secret` | Create encrypted Kubernetes sealed secrets for apps |

## Notes

The seal-secret skill can escalate to the Platform team via Jira if kubeseal is not available. For the escalation to work automatically, the user also needs the `stratus-builder` plugin installed (which provides the `request-platform-help` skill). Without it, the user will be given manual instructions for creating a PLAT ticket.
