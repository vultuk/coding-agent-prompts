Using the gh command, identify and fix any failing GitHub Actions workflows for the current repository.

1. **Detect the latest failing run**
   - Use `gh run list --limit 10 --json databaseId,name,conclusion,headSha,headBranch`  
     to find the most recent run with `"conclusion": "failure"`.
   - Store its ID as `<RUN_ID>`.

2. **Pull the failing logs**
   - Retrieve the logs for that run using:  
     `gh run view <RUN_ID> --log > action_failure.log`
   - Analyse the log output to determine the cause of failure (e.g. build error, lint issue, missing dependency, syntax error, or test failure).

3. **Diagnose and repair**
   - Based on the log details, inspect relevant files and code paths.
   - Apply necessary fixes to resolve the failure.  
     Examples include:
       • Updating syntax or import paths.  
       • Adjusting configuration or environment variables.  
       • Fixing tests or build commands.

4. **Verify the fix**
   - Commit and push your changes with:  
     `git commit -am "Fix GitHub Action failure: <summary>" && git push`
   - Trigger a re-run of the workflow to confirm success:  
     `gh workflow run <WORKFLOW_NAME> --ref <BRANCH_NAME>`

5. **Report progress**
   - Post a summary comment on the relevant PR or commit using:  
     `gh pr comment <PR_NUMBER> -b "<summary_of_fix>"`
     including:
       • The error identified.  
       • The exact change implemented.  
       • Confirmation that the workflow re-run has passed (or next steps if still failing).

Ensure that:
- The AI fully understands the failure context before changing code.  
- Only meaningful code fixes are committed (no superficial edits).  
- Every fix is validated by a successful GitHub Actions run.
