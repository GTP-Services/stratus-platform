---
name: seal-secret
description: Create encrypted Kubernetes sealed secrets for internal tools. Guides non-technical users through the process of sealing API keys and other secrets. Falls back to Platform team ticket if kubeseal is not available.
triggers: [seal secret, create secret, add api key, encrypt secret]
---

<skill>

<purpose>
Guide user through creating a K8s sealed secret. Sealed secrets are encrypted so they can be safely stored in Git. Only the cluster can decrypt them.
</purpose>

<workflow>

<step name="check-kubeseal">
Check whether kubeseal is installed:

```bash
kubeseal --version
```

If NOT installed, use AskUserQuestion with these options:
- "Install kubeseal (takes 1 minute)" — provide `brew install kubeseal` and guide them through it
- "I can't install software — open a Platform ticket instead" — invoke the `request-platform-help` skill
</step>

<step name="identify-secrets">
Use AskUserQuestion to ask what secrets are needed. Present options (allow multiple selections):
- Claude/Anthropic API key
- Database connection string
- Other API key

For each secret selected, ask: do they have the value now, or do they need Platform to generate it?

If they need a secret generated (e.g., they don't have a Claude API key yet), invoke `request-platform-help` to open a PLAT ticket for the key generation before proceeding.
</step>

<step name="create-env-files">
Create `.env.example` with placeholder values for each secret identified. Example:

```
ANTHROPIC_API_KEY=your-api-key-here
DATABASE_URL=your-connection-string-here
```

Verify `.gitignore` contains `.env`:

```bash
grep -q "^\.env$" .gitignore || echo ".env" >> .gitignore
```

Confirm `.gitignore` is updated before proceeding.

Use AskUserQuestion with:
- "I've added my values to .env — continue"
- "I need help with this step"
</step>

<step name="seal-secrets">
Read `app.yaml` to extract `project-name` and `namespace`. Then create the K8s secret and seal it:

```bash
kubectl create secret generic <project-name>-secrets \
  --from-env-file=.env \
  --namespace=<namespace> \
  --dry-run=client \
  -o yaml | \
kubeseal \
  --controller-name=sealed-secrets \
  --controller-namespace=sealed-secrets \
  --format=yaml \
  > k8s/sealedsecret.yaml
```

Replace `<project-name>` and `<namespace>` with values from `app.yaml`.

If kubeseal fails for any reason, do not attempt to debug it — offer to invoke `request-platform-help` to create a PLAT ticket.
</step>

<step name="update-config">
Update `app.yaml` to add the secrets reference:

```yaml
env_from_secrets:
  - <project-name>-secrets
```

Update `k8s/kustomization.yaml` to include the sealed secret in the resources list:

```yaml
resources:
  - sealedsecret.yaml
```
</step>

<step name="verify">
Confirm the output looks correct:

```bash
cat k8s/sealedsecret.yaml | head -20
```

Check that:
- `kind` is `SealedSecret`
- The name matches `<project-name>-secrets`
- The namespace is correct
- The `encryptedData` block contains encrypted values (not your real values)

Remind the user: "Your secret is now encrypted and safe to commit. The .env file with your real values should NEVER be committed."
</step>

</workflow>

<constraints>
- NEVER display or log actual secret values
- NEVER commit .env files
- ALWAYS verify .gitignore before proceeding past create-env-files
- ALWAYS use --dry-run=client (never create real secrets directly in the cluster)
- If kubeseal fails, offer to create a PLAT ticket rather than debugging
</constraints>

</skill>
