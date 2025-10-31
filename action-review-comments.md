Using the gh command, review all comments on the current PR thoroughly.

1. **Fetch all review comments**
   - Use `gh pr view --comments` to list every review comment on the PR.

2. **For each comment:**
   - Analyse the text and decide whether code changes are required.
   - If fixes are required:
     • Apply the changes locally and commit them.
     • Generate a short AI reply acknowledging the fix, e.g.  
       “✅ Fixed: Updated logic as suggested — added null check to prevent crash.”
   - If no fix is needed:
     • Generate a polite AI reply explaining why, e.g.  
       “ℹ️ No change required — behaviour already matches expected outcome.”
   - **If the suggestion is valuable but out of scope for this PR:**
     • Automatically create a well-worded GitHub issue using the `gh issue create` command, summarising the suggestion, rationale, and potential implementation approach, e.g.  
       `gh issue create -t "Enhancement: Add caching layer to optimise performance" -b "Suggested during PR review. This improvement is out of scope for the current PR but would enhance efficiency. Proposed approach: introduce Redis-based caching in the data access layer."`
     • Reply to the comment thread with a link to the created issue, e.g.  
       “📝 Good suggestion — tracked separately as [#123](<issue_link>).”

   - Post the AI-generated reply **to the specific comment thread** using:  
     `gh api repos/:owner/:repo/pulls/comments/:comment_id/replies -f body="<auto_reply_text>"`

3. **Mark the corresponding review thread as resolved**
   - Use the GraphQL API:  
     `gh api graphql -f query='mutation { resolveReviewThread(input:{threadId:"<THREAD_ID>"}){ pullRequestReviewThread { id isResolved } } }'`

   - You can retrieve `threadId` values via:  
     `gh api graphql -f query='{ repository(owner:"<OWNER>", name:"<REPO>"){ pullRequest(number:<PR_NUMBER>){ reviewThreads(first:100){ nodes { id comments(first:1){ nodes { id body author { login } } } } } } } }'`

4. **Post a final summary and approve**
   - Post a PR summary comment with:  
     `gh pr comment <PR_NUMBER> -b "<summary>"`  
     summarising:
       • The total comments reviewed.  
       • Changes made.  
       • Any feedback left unaddressed and why.  
       • Any new issues created for out-of-scope suggestions.
   - Approve and submit the review:  
     `gh pr review <PR_NUMBER> --approve -b "All review comments have been addressed and resolved."`

Ensure every comment receives an appropriate AI-generated reply before being resolved.
