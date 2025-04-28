# commit-experiment

### Github action to verify the PR title

* [pr-title-check.yml](https://github.com/mikelabraca/commit-experiment/blob/260e5ea52b53d781b80bc03fb19903f805b09985/.github/workflows/pr-title-check.yml)
```
# .github/workflows/pr-title-check.yml
name: PR Title Check

on:
  pull_request:
    types: [opened, edited, synchronize, reopened]

jobs:
  validate-pr-title:
    runs-on: ubuntu-latest
    steps:
      - name: Check PR title
        env:
          PR_TITLE: ${{ github.event.pull_request.title }}
          HEAD_REF: ${{ github.head_ref }}
        run: |
          echo "Checking PR Title: $PR_TITLE"
          echo "Branch Name: $HEAD_REF"
          # Allow specific branches to skip the check
          if [[ "$HEAD_REF" =~ ^(dependabot|dependencies|maintenance)/ ]]; then
            echo "Branch is allowed to skip PR title validation."
            exit 0
          fi
          # Validate the PR title format
          if [[ ! "$PR_TITLE" =~ ^\[[A-Z]+-[0-9]+\]\ .+ ]]; then
            echo "::error ::PR title must start with [JIRA-ID] followed by a space and some text."
            exit 1
          fi
          echo "PR title is valid."
```

### Git hook to add Jira Ticket Id as a prefix

This will be need to be added to the `commit-msg` file.
```
#!/bin/sh

BRANCH_NAME=$(git symbolic-ref --short HEAD)
JIRA_TICKET=$(echo "$BRANCH_NAME" | grep -oE '[A-Z]+-[0-9]+')

if [ ! -z "$JIRA_TICKET" ]; then
  COMMIT_MSG_FILE=$1
  ORIGINAL_MSG=$(cat "$COMMIT_MSG_FILE")

  # Avoid adding duplicate ticket prefix
  if ! echo "$ORIGINAL_MSG" | grep -q "$JIRA_TICKET"; then
    echo "[$JIRA_TICKET] $ORIGINAL_MSG" > "$COMMIT_MSG_FILE"
  fi
fi
```

### Repo settings changes

Changes needed to use the PR title as a commit message:
![settings-pull-request](https://github.com/user-attachments/assets/4cb9f995-ff65-4db0-9d45-dd9c93e1dc8b)

Then, when merging the PR, the merge preview will appear like this:
![pull-request-merge-preview](https://github.com/user-attachments/assets/289ffaf1-6875-426d-8bde-e26cbbcaf51d)

Changes needed to execute the Github action and be required to pass to be able to merge the PR:
![required-check](https://github.com/user-attachments/assets/4c9d74bd-a8ff-44fb-b01e-842f261b1cdf)



