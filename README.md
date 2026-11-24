# Cloud Deployment, Monitoring & AI in DevOps

## CMD:

```bash
py -m pip install streamlit
```

```bash
py -m streamlit --version
```

```bash
py -m streamlit run app.py
```

## AI-assisted troubleshooting activity

## Streamlit Deployment Troubleshooting

### Problem

My Streamlit application works locally, but the GitHub Actions deployment/workflow fails with this error:

```
(Line: 6, Col: 3): Unexpected value 'paths'
(Line: 7, Col: 5): A sequence was not expected
```

### 1. Explanation in simple terms

This error means GitHub Actions could not parse the workflow configuration. YAML relies on indentation to define which settings belong to which section. The likely issue is that the `paths` filter was placed at the wrong indentation level instead of inside the `push` or `pull_request` event.

### 2. Likely causes: evidence vs. guesses

**Evidence from the error**
- The validation message identifies `paths` as an unexpected value.
- The error occurs while GitHub Actions validates the workflow configuration, before the job can run.

**Likely cause**
- The `paths` filter is incorrectly nested/aligned in the workflow YAML. GitHub Actions expects it inside an event such as `push` or `pull_request`.

**Other possibilities (not confirmed by this error)**
- Another workflow file may have a similar YAML indentation problem.
- Other event keys or filters may be incorrectly nested.
- The app could have separate dependency or runtime problems, but this message does not establish those.

### 3. Corrective actions

1. Open `.github/workflows/streamlit-ci.yml`.
2. Nest `paths` beneath `push` (and beneath `pull_request`, if used).
3. Check YAML indentation: use spaces, not tabs.
4. Save, commit, and push the corrected workflow.
5. Use this correctly structured trigger example:

```yaml
name: Streamlit CI

on:
  push:
    branches:
      - main
    paths:
      - "AISE_Cloud_Deployment_Practical/**"
      - ".github/workflows/**"
  pull_request:
    branches:
      - main
    paths:
      - "AISE_Cloud_Deployment_Practical/**"
      - ".github/workflows/**"
  workflow_dispatch:
```

### 4. How to verify the fix

1. Open the repository's **Actions** tab.
2. Select the **Streamlit CI** workflow and open the newest run.
3. Confirm the workflow is accepted and starts (the YAML validation error should be gone).
4. Review each step—checkout, Python setup, dependency installation, Python compilation, and Streamlit smoke test.
5. Treat the fix as confirmed only when the workflow run completes successfully (green **Success**). If it fails later, inspect the failed step's log and troubleshoot that separate error.

**Testing status:** The corrected workflow YAML has been checked locally for parsing, but a successful live GitHub Actions run has not yet been verified. Do not claim the fix is fully tested until the Actions run passes.
