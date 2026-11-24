# AI-Enabled Software Engineering — Unit 5, Day 3 Practical Guide

## Problem Statement

My Streamlit application runs successfully on my local computer, but the GitHub Actions deployment/workflow fails with this YAML validation error:

```text
(Line: 6, Col: 3): Unexpected value 'paths'
(Line: 7, Col: 5): A sequence was not expected
```

## 1. Explanation in Simple Terms

GitHub Actions expects workflow settings to be nested in a specific YAML structure. The `paths` filter was placed at the wrong indentation level: it was outside the `push` event, instead of being nested inside it.

YAML indentation defines which settings belong to which event. Because `paths` was in the wrong place, GitHub Actions did not recognize the workflow structure and rejected it before running the job.

## 2. Likely Causes — Evidence vs. Guesses

### Evidence from the workflow

- The reported validation error specifically points to `paths` as an unexpected value.
- In the workflow, `paths` was aligned with `push`, rather than nested under `push`.
- YAML uses indentation to express parent-child relationships, so this is a structural error.

### Other possible causes (not established by this error)

- A different workflow file may contain the same indentation problem.
- A workflow edit may have introduced a misplaced event filter.
- A deployment workflow may have a separate configuration or runtime issue.

The log confirms a YAML/workflow validation problem. It does **not**, by itself, prove that the Streamlit app, its dependencies, or the hosting configuration are broken.

## 3. Corrective Actions

### Step 1 — Put event filters under the event

For a push trigger, both `branches` and `paths` must be inside `push`. For a pull request, its filters must be under `pull_request`.

Correct structure:

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

### Step 2 — Check indentation carefully

- `push` and `pull_request` are children of `on`.
- `branches` and `paths` are children of `push` (or `pull_request`).
- Each path pattern is an item in a list and starts with a dash.
- Use spaces, not tabs, for YAML indentation.

### Step 3 — Save and commit the workflow

Ensure the corrected workflow is saved in the repository under:

```text
.github/workflows/streamlit-ci.yml
```

### Step 4 — Push the corrected configuration

Push the change to the `main` branch. Since the workflow watches the app folder and workflow folder, a change to this workflow should trigger it.

## 4. How to Verify the Fix

1. Open the repository's **Actions** tab.
2. Select the **Streamlit CI** workflow.
3. Confirm a new run is created after the workflow change is pushed.
4. Open the run and check each step:
   - Checkout repository
   - Set up Python
   - Install dependencies
   - Compile Python files
   - Smoke test Streamlit app
5. Confirm the run finishes with a green check (Success).
6. If it fails, open the failing step and read its log. A YAML validation error occurs before jobs run; a dependency, syntax, or smoke-test error occurs later and needs a different diagnosis.

### Important testing note

A corrected YAML file is not considered proven just because it looks right. The fix is verified only after GitHub Actions accepts the workflow and a run completes successfully. This guide documents the correction and the verification procedure; a successful live Actions run must be checked separately.

---

## Run Streamlit Locally

From the app directory:

```bash
py -m pip install -r requirements.txt
py -m streamlit run app.py
```

