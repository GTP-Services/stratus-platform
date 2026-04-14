# Stratus Platform

A Claude Code plugin providing platform utilities for GTP-Services teams.

## Installation

### Via Stratus Marketplace (recommended)

```bash
claude plugin marketplace add GTP-Services/stratus-marketplace
claude plugin install stratus-platform
```

### Direct Install

```bash
claude plugin install GTP-Services/stratus-platform
```

## Usage

```
/stratus-platform:seal-secret       # Create an encrypted secret for your app
```

## Note on Escalation

The seal-secret skill can open Platform team tickets when kubeseal isn't available. This requires the `request-platform-help` skill from the `stratus-builder` plugin. If not installed, you'll get manual instructions for creating the ticket in Jira.
