---
name: dev-helper
description: Reviews pull requests and also deploys to staging and production, and also writes release notes, and also runs database migrations, and also updates the on-call runbook, and also triages incoming bug reports. Use when you want help with almost anything in the dev workflow.
---

# Dev Helper

Does most of the team's dev workflow in one place.

## Instructions

### Reviewing PRs
1. Read the diff, flag error handling and missing tests.

### Deploying
1. Build the image, push, run the deploy script for staging or prod.

### Release notes
1. Read merged PRs since the last tag, group by type, write the changelog.

### Database migrations
1. Generate the migration, review it, apply with backup.

### On-call runbook
1. Update the runbook when an alert changes.

### Bug triage
1. Read the new issue, label it, assign severity, route to a team.

## Examples

User says: "help with the dev stuff" - pick whichever section applies.
