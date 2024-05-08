# Renzo Protocol Audit - Validation Repo

Unless otherwise discussed, this repo will be made public after audit completion, sponsor review, judging, and issue mitigation window.

**Contributors to this repo:** prior to report publication, please review the [Agreements & Disclosures](https://github.com/code-423n4/2024-04-renzo-validation/issues/1) issue.

## Instructions for Validators

### Acting on an issue

When a validator has been assigned to an issue, they have four different actions they can take.
Each of these is performed by typing a command into the issue's comments and sending it. Each
command must be prefixed by the `trigger` phrase the bot was configured for. In production, this
is `@howlbot` and the remainder of this document will assume that value for simplicity, though it
may be different in different environments.

#### `@howlbot accept`

Accepts an issue as valid and forwards it to the corresponding `-findings` repository.

1. Label the issue `sufficient quality report`
2. Open a new issue with the same contents in the `-findings` repository
3. Find the JSON file corresponding to the issue
4. Modify the JSON file to add the `validatedIssueId` and `validatedIssueUrl` fields pointing to the newly created issue
5. Update the JSON file in the `-validation` repository
6. Create a new JSON file in the `-findings` repository with `originalIssueId` and `originalIssueUrl` fields pointing to the issue in the `-validation` repository
7. Close the issue

#### `@howlbot reject`

Rejects an issue.

1. Label the issue `insufficient quality report`
2. Close the issue

#### `@howlbot skip`

Allows a validator to flag an issue as having *not* been reviewed, while allowing them to move on to a different issue.

1. Label the issue `unknown`
2. Unassign the current validator

#### `@howlbot edit`

Allows a validator to modify the contents of a submission.

This command accepts a parameter in the form of the new content for the submission, to use it:

```
@howlbot edit

this is where my new submission content would go. this will replace the current content of the issue
as well as adding a label to the issue, but it will not automatically accept it, you'll still have
to do that separately
```

1. Replace the contents of the submission body
2. Label the issue `improved`

### Receiving additional assignments

After a validator has completed, or skipped, an issue they will be automatically assigned *one* new
issue. If the issue had been previously marked as `unknown` and is either accepted or rejected, a
bonus of three additional issues is included and the validator will receive a total of *four* new
assignments.
