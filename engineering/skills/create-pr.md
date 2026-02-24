---
name: create-pr
description: Create or update a GitHub Pull Request from a developer’s perspective with clear, descriptive QA testing steps and automatically apply the `qa-needed` label so the QA automation pipeline can process it. Use when the user asks to create a PR, open a PR, draft a PR, add QA test steps, or prepare work for QA.
---

# Create PR (QA-Ready, Dev-Focused)

This skill prepares a Pull Request exactly how a developer would hand completed work over to QA.

The developer focuses only on:
- What the ticket is about
- What changed
- How QA can test it

This skill ensures:
- The PR body follows a consistent structure
- Testing steps are explicit and easy to follow
- The `qa-needed` label is applied
- The PR is ready for the automated QA pipeline

---

## When to Use

- Use this skill when a developer has completed a ticket and needs to open a PR.
- Use this skill when a PR exists but is missing proper QA testing steps.
- Use this skill when QA instructions are vague and need to be structured clearly.
- Use this skill when a PR should enter the QA pipeline.

---

## Instructions

### 1. Gather Required Information

Before creating or updating the PR, ensure you have:

- Ticket / feature name or purpose
- Short summary of what was implemented
- Bullet list of changes
- Where the feature can be accessed (route, screen, navigation path)
- Any setup requirements (feature flags, account roles, seed data)
- Any important edge cases that should be validated

Use the ask questions tool if clarification is needed.

---

### 2. Create or Update the PR

If a PR does not exist:
- Create a new PR targeting the correct base branch.

If a PR already exists:
- Update the PR body so it matches the required structure below.
- Replace outdated QA steps if necessary.
- Do not duplicate sections.

---

### 3. Required PR Body Structure

The PR must contain exactly the following sections:

## Summary
Clear explanation of what this ticket does and why.

## Changes
- Bullet list of implemented changes
- Include only relevant technical notes QA should know

## QA Instructions

### Setup (only if needed)
- [ ] Feature flags enabled
- [ ] Required test account / role
- [ ] Special environment setup

### Test Cases

1. Happy Path
   - Steps:
     1) Step one
     2) Step two
   - Expected:
     - Expected result

2. Edge Case / Negative Case
   - Steps:
     1) Step one
   - Expected:
     - Expected result

### Regression Checklist
- [ ] App/service loads correctly
- [ ] No crashes
- [ ] Related flows still function
- [ ] No obvious UI regressions

---

### 4. How to Write Good QA Instructions

QA Instructions must be descriptive and guide QA step-by-step to the feature.

They should clearly explain:
- Where the feature is located
- How to navigate to it
- What action to perform
- What exact result should occur

Avoid vague instructions like:
- "Test the settings feature"
- "Verify it works"

Instead, write instructions like this:

### Example 1 – Feature in Settings Screen

Bad:
- Go to settings and test the new toggle.

Good:
1. Launch the app and log in as a standard user.
2. From the home screen, tap the **Settings icon** in the top-right corner.
3. Scroll to the **“Notifications”** section.
4. Locate the new toggle labeled **“Enable Smart Alerts”**.
5. Turn the toggle ON.
   - Expected:
     - The toggle remains enabled after navigating away and back.
     - A confirmation toast appears saying “Smart Alerts Enabled”.
6. Close and reopen the app.
   - Expected:
     - The toggle remains enabled.

Edge Case:
1. Disable internet connection.
2. Attempt to enable the toggle.
   - Expected:
     - An error message appears.
     - The toggle returns to OFF state.

---

### Example 2 – New Button in a Flow

Bad:
- Test the new export button.

Good:
1. Navigate to the Dashboard screen.
2. Select any existing report.
3. Click the new **“Export as PDF”** button located in the top-right.
   - Expected:
     - A loading spinner appears.
     - A PDF file downloads automatically.
4. Open the downloaded file.
   - Expected:
     - The file contains the correct report data.
     - No formatting issues are present.

---

### What Good QA Instructions Always Include

- Exact navigation path (where to tap/click)
- Clear UI labels as they appear in the app
- Expected behavior after each important action
- At least one edge or failure case
- Any required role, account, or configuration

QA should never need to guess where the feature is or what success looks like.

---

### 5. Apply QA Label

- Ensure the PR has the label: `qa-needed`
- If it is missing, add it.

---

## Completion Checklist

Before finishing, confirm:

- PR includes Summary, Changes, and QA Instructions sections.
- QA Instructions contain clear navigation steps and expected results.
- At least one happy path is included; include an edge case when the change has meaningful failure modes.
- `qa-needed` label is applied.
- The PR is ready for the QA automation pipeline.