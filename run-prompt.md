Before executing the user-supplied prompt at the end of this message, first append a log entry to `.prompts/log.md`. If the path does not exist, create it.

Logging requirements:

1) Create the `.prompts/` directory if needed. Never overwrite `log.md`; always append.
2) Write a single entry per run using the exact Markdown template below.
3) Use an ISO‑8601 UTC timestamp (e.g. 2025-09-04T12:34:56Z).
4) Determine the user as follows:
   - If inside a Git repository, prefer `git config --get user.name` and `git config --get user.email` (if set).
   - If not available, fall back to the operating system user (e.g. `$USER`, `whoami`).
5) Preserve the prompt text verbatim, including all whitespace and newlines.
6) Use a FOUR‑backtick Markdown fence for the prompt block to avoid collisions if the prompt contains triple backticks.
7) If logging fails for any reason, stop and surface the error rather than executing the prompt.

Markdown entry template (fill in the bracketed placeholders exactly once per run):

### [TIMESTAMP_UTC]
- user: [DISPLAY_NAME and optionally <email>]
````text
$ARGUMENTS
````

After the log entry has been appended successfully, execute exactly the instructions below.

$ARGUMENTS