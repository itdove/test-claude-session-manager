# CS Tool Usage Guide for Claude

This file provides comprehensive instructions for using the `cs` tool (Claude Session Manager) within Claude Code sessions. It is automatically loaded as context when you open a session via `cs open`.

**What is cs?** The `cs` tool is a Claude Code session manager that helps organize development work, track time, and optionally integrate with JIRA for ticket management.

**When to use this guide:** This file is automatically read when sessions are opened. Refer to it when you need to:
- Create or update JIRA tickets
- Manage sessions and workflows
- Work with git branches and PRs/MRs
- Understand cs tool commands and best practices

**Note**: Examples use generic placeholders (PROJECT, YourWorkstream). Actual values are configured in `~/.claude-sessions/config.json`.

---

## Git Workflow

**IMPORTANT**: Never commit directly to the `main` branch. Always create a feature branch before making any commits.

### Creating Branches and Pull Requests

1. **Update main branch** before creating a new branch:
   ```bash
   git checkout main
   git pull origin main
   ```
   **IMPORTANT**: Always ensure your main branch is up-to-date before creating a new feature branch.

2. **Create a branch** from the updated main branch:
   ```bash
   git checkout -b <JIRA-KEY>-<short-description>
   ```
   Example: `git checkout -b PROJECT-12345-fix-validation`

3. **Make your changes** and commit them to the branch

4. **Push the branch** to remote:
   ```bash
   git push -u origin <branch-name>
   ```

5. **Create a draft PR/MR** using the template from your project
   - For GitHub repositories: Use `gh` CLI
   - For GitLab repositories: Use `glab` CLI
   - Or use `cs complete` which can create PRs/MRs automatically

### Updating Existing Pull/Merge Requests

**IMPORTANT**: When pushing additional commits to a branch that already has an open PR/MR, you MUST update it to reflect the changes:

1. **Update the PR/MR title** if the scope or focus of the changes has evolved
2. **Update the PR/MR body** to document what functionality was:
   - Added (new features or capabilities)
   - Changed (modifications to existing functionality)
   - Deleted (removed features or code)

**For GitLab (using glab)**:
```bash
# Update MR title
glab mr update <MR-NUMBER> --title "Updated title reflecting all changes"

# Update MR description
glab mr update <MR-NUMBER> --description "$(cat <<'EOF'
## Description
- Added: New validation for instance names
- Changed: Refactored error handling
- Deleted: Deprecated legacy endpoints

...rest of your MR description...
EOF
)"
```

**For GitHub (using gh)**:
```bash
# Update PR title
gh pr edit <PR-NUMBER> --title "Updated title reflecting all changes"

# Update PR body
gh pr edit <PR-NUMBER> --body "$(cat <<'EOF'
## Description
...your updated description...
EOF
)"
```

---

## Session Management with cs Tool

The `cs` tool manages Claude Code sessions, allowing you to:
- Create and resume focused work sessions
- Track time spent on tasks
- Organize work by JIRA tickets or custom names
- Auto-generate PR descriptions and JIRA updates
- Export session summaries

### Basic Session Workflow (No JIRA Required)

```bash
# Create a new session for any work
cs new --name "redis-performance" --goal "Test Redis caching performance"

# Work in Claude Code...

# Add progress notes
cs note redis-performance "Benchmark results: 10k req/s"

# Complete and export session
cs complete redis-performance
```

---

## JIRA Integration (Optional)

**IMPORTANT**: For projects using JIRA, ALL JIRA operations MUST be performed using `cs jira` commands, which use the JIRA REST API internally.

**DO NOT USE**:
- The standalone `jira` CLI tool (unreliable visibility control, validation issues)
- Direct curl commands (use `cs jira` instead for automatic authentication and field handling)
- The JIRA web interface for automated operations

### Environment Variables

Required for JIRA integration:
```bash
export JIRA_API_TOKEN="your-personal-access-token"
export JIRA_AUTH_TYPE="Bearer"
```

### Configuring cs Tool

Set your JIRA project, workstream, and affected version (one-time setup):

```bash
# Set JIRA project key (replace PROJECT with your project key)
cs config set-project PROJECT

# Set JIRA workstream (replace with your workstream name)
cs config set-workstream YourWorkstream

# Set affected version for bugs (replace with your project's affected version)
cs config set-affected-version <affected-version>
# Example: cs config set-affected-version ansible-saas-ga

# Refresh JIRA field mappings (optional, for custom fields)
cs config refresh-jira-fields
```

### Configuring Context Files

The cs tool supports configurable context files that are automatically included in the initial prompt when creating or opening sessions. This helps Claude understand your project's context, standards, and architecture.

**Default Context Files** (always included):
- `AGENTS.md` - Agent-specific instructions
- `CLAUDE.md` - Project guidelines and standards

**Additional Context Files** (configurable):

```bash
# List all configured context files
cs config context list

# Add a local file (Claude will use Read tool)
cs config context add ARCHITECTURE.md "system architecture"

# Add a URL (Claude will use WebFetch tool)
cs config context add https://github.com/org/repo/blob/main/STANDARDS.md "coding standards"

# Remove a context file
cs config context remove ARCHITECTURE.md

# Reset to defaults (removes all configured files)
cs config context reset
```

**How It Works:**
- When you create or open a session, the initial prompt includes instructions to read all configured context files
- Default files (AGENTS.md, CLAUDE.md) are always included first
- Additional configured files are included after defaults
- Claude automatically detects whether to use Read (local files) or WebFetch (URLs) based on the path

**Example:**
```bash
# Add project-specific context files
cs config context add ARCHITECTURE.md "system architecture"
cs config context add DESIGN.md "design docs"
cs config context add https://github.com/your-org/your-repo/blob/main/STANDARDS.md "coding standards"

# List to verify
cs config context list
```

This ensures Claude has all necessary project context before starting work on any task.

**IMPORTANT - Affected Version for Bugs:**

When creating Bug type issues:
1. **Configure it once**: Run `cs config set-affected-version <affected-version>` (replace with your project's version)
2. **Automatic usage**: After configuration, `cs jira create bug` uses it automatically
3. **If not configured**: The command will prompt you to enter it and save it for future use
4. **To override**: Use `--affected-version <value>` flag when creating a specific bug

**Agent Instructions**: Before creating a bug, check if affected version is configured in `~/.claude-sessions/config.json`. If NOT configured, ASK the user "What is the affected version for this bug?" and suggest using `cs config set-affected-version <affected-version>` to set it permanently.

### Viewing JIRA Issues

```bash
# View any JIRA ticket in Claude-friendly format (replace with your ticket key)
cs jira view PROJECT-12345
```

**Always use `cs jira view` instead of curl** - it handles authentication automatically and provides reliable output.

### Creating JIRA Issues with Codebase Analysis (Recommended)

For creating well-researched JIRA tickets based on codebase analysis, use `cs jira new`. This creates an analysis-only session that:
- Prevents accidental code modifications
- Skips branch creation automatically
- Guides you through codebase exploration
- Helps write detailed, accurate JIRA tickets

```bash
# Create a story with codebase analysis
cs jira new story --parent PROJECT-1234 --goal "Add retry logic to subscription API"

# Create a bug with analysis
cs jira new bug --parent PROJECT-1234 --goal "Fix timeout in backup operation"

# Create a task
cs jira new task --parent PROJECT-1234 --goal "Update documentation for new feature"
```

**What happens:**
1. Creates an analysis-only session (session_type="ticket_creation")
2. Launches Claude with READ-ONLY constraints
3. You analyze the codebase to understand implementation details
4. You create the JIRA ticket using `cs jira create` command
5. **IMPORTANT**: When creating the ticket, use `--parent <parent>` to link it to the parent

**Example workflow:**
```bash
# 1. Start analysis session
cs jira new story --parent PROJECT-1234 --goal "Add retry logic to subscription API"

# 2. Claude analyzes codebase (read-only)
# - Searches for relevant files
# - Understands existing patterns
# - Identifies integration points

# 3. Claude creates the JIRA ticket
cs jira create story \
  --summary "Add retry logic to subscription API" \
  --parent PROJECT-1234 \
  --description "..." \
  --acceptance-criteria "..."

# 4. Exit Claude and session completes
```

**Note on Parent Mapping:**
- The `--parent` value from `cs jira new` is used with `--parent` in `cs jira create`
- Config patches (004-parent-field-mapping.json) automatically map `--parent` to the correct JIRA field based on issue type:
  - Story/Task/Bug → Epic Link field (via field_mappings['epic_link'])
  - Sub-task → Parent field (standard JIRA system field)

**Benefits:**
- No accidental code changes during research
- Better informed JIRA tickets with accurate estimates
- Automatic session name generation
- Persistent constraints when reopening

### Creating JIRA Issues Directly (Without Analysis)

For quick ticket creation without codebase analysis:

#### Create a Bug

```bash
cs jira create bug \
  --summary "Customer backup fails with timeout" \
  --priority Major \
  --parent PROJECT-1234 \
  --description-file /tmp/bug_description.txt
```

**Note**: Replace `PROJECT-1234` with your epic key.

Bug description template:
```
*Description*

_<what is happening, why are you requesting this update>_

*Steps to Reproduce*

_<list explicit steps to reproduce>_

*Actual Behavior*

_<what is currently happening>_

*Expected Behavior*

_<what should happen?>_

*Additional Context*

<_Provide any related communication on this issue._>
```

#### Create a Story

```bash
# Interactive mode (opens editor for description)
cs jira create story \
  --summary "Implement backup and restore feature" \
  --priority Major \
  --parent PROJECT-1234 \
  --interactive

# With description from file
cs jira create story \
  --summary "Implement backup feature" \
  --parent PROJECT-1234 \
  --description-file /tmp/story.txt

# Inline description with acceptance criteria
cs jira create story \
  --summary "Implement backup feature" \
  --parent PROJECT-1234 \
  --acceptance-criteria "- Users can create backups\n- Backups are validated" \
  --description "h3. *User Story*\n\nAs a user I want..."
```

Story description template:
```
h3. *User Story*

Format: "as a <type of user> I want <some goal> so that <some reason>"

h3. *Supporting documentation*

<include links to technical docs, diagrams, etc>
```

#### Create a Task

```bash
cs jira create task \
  --summary "Update backup documentation" \
  --parent PROJECT-1234 \
  --description "h3. *Problem Description*\n\nUpdate docs to reflect new feature"
```

Task description template:
```
h3. *Problem Description*

<what is the issue, what is being asked, what is expected>

h3. *Supporting documentation*
```

#### JIRA Issue Templates

**IMPORTANT**: All JIRA issue descriptions MUST use Jira Wiki markup syntax, NOT Markdown.

Jira Wiki markup ensures proper rendering in the JIRA UI. Key differences from Markdown:
- Headers: Use `h2.` or `h3.` instead of `##` or `###`
- Bold: Use `*text*` instead of `**text**`
- Italic: Use `_text_` instead of `*text*`
- Code blocks: Use `{code:bash}...{code}` instead of triple backticks
- Inline code: Use `{{code}}` instead of backticks

The following templates MUST be used for the description when creating new issues. These templates ensure consistency across all JIRA tickets.

**Epic Template:**
```
h2. *Background*

{color:#0747a6}_Initial completion during New status and then remove this blue text._{color}

<fill out any context, value prop, description needed>

h2. *User Stories*

{color:#0747a6}_Initial completion during New status and then remove this blue text._{color}

Format: "as a <type of user> I want <some goal> so that <some reason>"

h2. *Supporting documentation*

{color:#0747a6}_Initial completion during New status and then remove this blue text._{color}

<include links to technical docs, diagrams, etc>
```

**Spike Template:**
```
h3. *User Story*

Format: "as a <type of user> I want <some goal> so that <some reason>"

h3. *Supporting documentation*

<include links to technical docs, diagrams, etc>
```

**Note**: Each Spike must be linked to an Epic.

**Story Template:**
```
h3. *User Story*

Format: "as a <type of user> I want <some goal> so that <some reason>"

h3. *Supporting documentation*

<include links to technical docs, diagrams, etc>
```

**Note**: Each Story must be linked to an Epic.

**Bug Template:**
```
*Description*

_<what is happening, why are you requesting this update>_

*Steps to Reproduce*

_<list explicit steps to reproduce, for docs bugs, include the error/issue>_

*Actual Behavior*

_<what is currently happening, for docs bugs include link(s) to relevant section(s)>_

*Expected Behavior*

_<what should happen? for docs bugs, provide suggestion(s) of how to improve the content>_

*Additional Context*

<_Provide any related communication on this issue._>
```

**Task Template:**
```
h3. *Problem Description*

<what is the issue, what is being asked, what is expected>

h3. *Supporting documentation*
```

#### Create with Session

Automatically create a cs session for the newly created issue:

```bash
cs jira create story \
  --summary "New feature" \
  --parent PROJECT-1234 \
  --create-session
```

This creates the JIRA issue and immediately starts a session for development work.

**Note**: This is a convenience feature that combines `cs jira create` + `cs open` in one command.

#### Create with Custom Fields

Use the `--field` option to set any JIRA custom field:

```bash
# Create bug with custom severity and size
cs jira create bug \
  --summary "Security vulnerability in auth" \
  --priority Critical \
  --field severity=Critical \
  --field size=L \
  --parent PROJECT-1234

# Multiple custom fields
cs jira create story \
  --summary "New API endpoint" \
  --priority Major \
  -f epic_link=PROJECT-1234 \
  -f story_points=5 \
  -f size=M
```

**Note**: Custom fields are discovered automatically on first use and cached for future commands.

**To see available custom fields**:
```bash
# View all available custom fields for creation
cs jira create story --help

# The --field option will show available fields dynamically
# Example output: "Available fields: acceptance_criteria, severity, size, story_points, etc."
```

#### Dynamic Custom Fields

Both `cs jira create` and `cs jira update` support dynamic custom field discovery:

**For `cs jira create`** (cached fields):
```bash
# View available creation fields (from cache)
cs jira create --help
# → Shows: --story-points, --severity, --size, etc.

# Use dynamic creation options
cs jira create bug --summary "Test" --severity Critical --size L

# Or use universal --field option
cs jira create bug --summary "Test" --field severity=Critical
```

**For `cs jira update`** (on-demand discovery):
```bash
# Discover what fields you can update for a specific issue
cs jira update PROJECT-12345 --help
# → Shows: --epic-link, --story-points, --sprint, --blocked, etc.
# → Fields are discovered fresh each time for that specific issue

# Use dynamic update options (shown in help)
cs jira update PROJECT-12345 --severity Critical --blocked True

# Or use universal --field option
cs jira update PROJECT-12345 --field epic_link=PROJECT-1234
```

**Key Differences:**
- **Create fields**: Cached in `~/.claude-sessions/config.json`, refreshed with `cs config refresh-jira-fields`
- **Update fields**: Discovered on-demand when viewing help (not cached), specific to each issue's current state

### Updating JIRA Issues

```bash
# Update description
cs jira update PROJECT-12345 --description "New description text"

# Update from file
cs jira update PROJECT-12345 --description-file /tmp/new_description.txt

# Update multiple fields
cs jira update PROJECT-12345 \
  --priority Major \
  --workstream YourWorkstream \
  --summary "Updated summary"

# Update acceptance criteria
cs jira update PROJECT-12345 \
  --acceptance-criteria "- Criterion 1\n- Criterion 2\n- Criterion 3"

# Update custom fields using --field option
cs jira update PROJECT-12345 --field severity=Critical --field size=L

# Add PR link (auto-appends to existing links)
cs jira update PROJECT-12345 --git-pull-request "https://github.com/org/repo/pull/123"

# Use dynamic options (discovered when viewing help with issue key)
cs jira update PROJECT-12345 --epic-link PROJECT-1234 --story-points 5
```

### Working with Sessions

#### Session Workflow with JIRA Integration

If your project uses JIRA for ticket tracking:

```bash
# 1. Sync assigned JIRA tickets
cs sync --sprint current

# 2. Open a session for a ticket (auto-loads JIRA context)
cs open PROJECT-12345

# 3. Work in Claude Code...
#    Claude automatically loads JIRA ticket details as context

# 4. Add progress notes
cs note PROJECT-12345 "Completed API implementation"

# 5. Complete the session (creates PR, updates JIRA)
cs complete PROJECT-12345
# - Prompts to commit uncommitted changes
# - Creates PR/MR with AI-generated summary
# - Adds session summary to JIRA
# - Transitions JIRA ticket status
```

#### Session Workflow Without JIRA

For experiments, spikes, or work not tracked in JIRA:

```bash
# 1. Create a new session
cs new --name "redis-cache-test" --goal "Test Redis performance"

# 2. Work in Claude Code...

# 3. Add progress notes
cs note redis-cache-test "Benchmark complete: 10k req/s"

# 4. Complete the session
cs complete redis-cache-test
# - Prompts to commit changes
# - Creates PR/MR
# - Exports session summary

# Optional: Link to JIRA later if needed
cs session link redis-cache-test --jira PROJECT-12345
```

#### Viewing Session Status

```bash
# View all active sessions
cs status

# View session history
cs list

# View specific session details
cs show PROJECT-12345
```

### Best Practices

#### General Session Management
1. **Use descriptive session names** - Makes it easier to find and resume work
2. **Add notes regularly** - Use `cs note` to track progress
3. **One session per feature/ticket** - Keeps context focused
4. **Let `cs complete` handle commits/PRs** - Automated workflow with AI-generated summaries

#### JIRA-Specific Best Practices
5. **Always use `cs jira view` to read tickets** - More reliable than curl
6. **Configure project/workstream once** - Use `cs config set-project` and `cs config set-workstream`
7. **Use `--interactive` for complex descriptions** - Opens editor for easier formatting
8. **Use `--create-session` when ready to start work** - Streamlines workflow from JIRA creation to coding
9. **Use `--field` for custom fields** - Works for any JIRA custom field, automatically discovered
10. **Discover available fields with --help**:
    - For creation: `cs jira create --help` (shows cached fields)
    - For updates: `cs jira update PROJECT-12345 --help` (discovers fields for that specific issue)

### Common Commands Reference

```bash
# Session Management (Core)
cs new --name "feature-name" --goal "Description"
cs open <session-name-or-jira-key>
cs note <session-name> "Progress update"
cs complete <session-name>
cs status
cs list
cs show <session-name>

# JIRA Integration (Optional)
cs config set-project PROJECT
cs config set-workstream YourWorkstream
cs config set-affected-version <affected-version>
cs config refresh-jira-fields
cs sync --sprint current

# JIRA Operations
cs jira view PROJECT-12345
cs jira create {bug|story|task} --summary "..." --parent PROJECT-1234
cs jira create bug --summary "..." --field severity=Critical --field size=L
cs jira update PROJECT-12345 --description "..."
cs jira update PROJECT-12345 --field severity=Critical
cs jira update PROJECT-12345 --git-pull-request "https://..."

# Get Help
cs --help
cs jira create --help                    # View creation fields (cached)
cs jira update PROJECT-12345 --help      # View editable fields (on-demand)
cs config --help
```

### Troubleshooting

#### General Issues

**Error: "cs: command not found"**
```bash
# Install cs with pip
pip install claude-session-manager

# Or reinstall
pip install --upgrade --force-reinstall claude-session-manager
```

**Session not found**
```bash
# List all sessions
cs list

# View session status
cs status
```

#### JIRA-Specific Issues

**Error: "project is required"**
```bash
# Set project in config (one-time setup)
cs config set-project PROJECT
```

**Error: "workstream is required"**
```bash
# Set workstream in config (one-time setup)
cs config set-workstream YourWorkstream
```

**Error: "JIRA_API_TOKEN not set"**
```bash
# Set API token in environment
export JIRA_API_TOKEN="your-personal-access-token"
export JIRA_AUTH_TYPE="Bearer"

# Add to shell profile for persistence
echo 'export JIRA_API_TOKEN="your-token"' >> ~/.zshrc
echo 'export JIRA_AUTH_TYPE="Bearer"' >> ~/.zshrc
```

**Custom fields not showing**
```bash
# Refresh field mappings from JIRA
cs config refresh-jira-fields

# Then check help again
cs jira create story --help
```

### Integration with Git Workflow

The `cs` tool integrates seamlessly with git:

```bash
# When opening a session, it can:
# - Auto-create a branch (e.g., PROJECT-12345-feature-name)
# - Pull latest changes from main
# - Set up git tracking

# When completing a session, it can:
# - Commit uncommitted changes with AI-generated message
# - Push branch to remote
# - Create PR/MR with proper template
# - Link PR to JIRA ticket
```

#### Branch Naming Convention

Format: `<JIRA-KEY>-<short-description>`
- Use lowercase with hyphens
- Keep the description concise but meaningful
- Examples:
  - `PROJECT-123-add-caching-layer`
  - `PROJECT-456-fix-timeout-issue`
  - `PROJECT-789-refactor-api-client`

#### Commit Message Format

When creating commits with AI assistance, follow this format:

```bash
git commit -m "$(cat <<'EOF'
Brief summary of changes (imperative mood, < 50 chars)

More detailed explanation if needed. Explain what and why, not how.
- Bullet points are acceptable
- Use present tense

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**IMPORTANT**:
- All commits made with AI assistance MUST include the `Co-Authored-By` field
- Format: `Co-Authored-By: <Name> <email>` (e.g., `Co-Authored-By: Claude <noreply@anthropic.com>`)

### GitHub and GitLab CLI Tools

**IMPORTANT**: Use the appropriate CLI tool based on your repository platform:

#### For GitHub Repositories (using `gh`)

**Installation:**
```bash
# macOS
brew install gh

# Linux (Debian/Ubuntu)
sudo apt install gh

# Other platforms: https://cli.github.com/
```

**Authentication:**
```bash
# Interactive authentication
gh auth login

# Or with a token
export GITHUB_TOKEN="your-github-token"
gh auth login --with-token < <(echo $GITHUB_TOKEN)
```

**Create Pull Request:**
```bash
gh pr create --draft --title "Title" --body "$(cat <<'EOF'
Jira Issue: https://<JIRA-URL>/<ISSUE-KEY>

## Description

[Describe your changes here]

Assisted-by: Claude (Anthropic)

## Testing
...

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

#### For GitLab Repositories (using `glab`)

```bash
# Installation
brew install glab

# Authentication (for Red Hat GitLab)
glab auth login --hostname gitlab.example.com

# Full authentication with explicit parameters
glab auth login --hostname gitlab.example.com \
  --api-host gitlab.example.com \
  --api-protocol https \
  --git-protocol git \
  -t $GITLAB_TOKEN

# Create Merge Request
glab mr create --draft --title "Title" --description "$(cat <<'EOF'
Jira Issue: https://<JIRA-URL>/<ISSUE-KEY>

## Description

[Describe your changes here]

Assisted-by: Claude (Anthropic)

## Testing
...

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**Important Notes:**
- For GitHub: Always use `gh pr create`
- For GitLab: Always use `glab mr create`
- Always include "Assisted-by: Claude (Anthropic)" when AI assistance is used
- Always include Co-Authored-By field in commit messages
- Use `--draft` flag to create draft PR/MR

**Determining git-protocol:**
```bash
# Check remote URL format
git remote -v

# If using git@gitlab.example.com:..., use --git-protocol git
# If using https://gitlab.example.com/..., use --git-protocol https
```

---

## Fetching Files from Private GitHub Repositories

**IMPORTANT**: When you need to fetch files from private GitHub repositories (such as AGENTS.md files or other documentation), you MUST use the GitHub REST API with authentication via the `GITHUB_TOKEN` environment variable.

### Using GitHub REST API to Fetch File Contents

The `GITHUB_TOKEN` environment variable should be set with a valid GitHub Personal Access Token that has access to private repositories.

**To fetch raw file contents:**

```bash
curl -H "Accept: application/vnd.github.raw" \
  -H "Authorization: Bearer ${GITHUB_TOKEN}" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/repos/{OWNER}/{REPO}/contents/{PATH}"
```

**Example - Fetching an AGENTS.md file:**

```bash
curl -H "Accept: application/vnd.github.raw" \
  -H "Authorization: Bearer ${GITHUB_TOKEN}" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/repos/YourOrg/your-repo/contents/AGENTS.md"
```

**Example - Fetching a file from a subdirectory:**

```bash
curl -H "Accept: application/vnd.github.raw" \
  -H "Authorization: Bearer ${GITHUB_TOKEN}" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/repos/YourOrg/your-repo/contents/docs/architecture/README.md"
```

**Important Notes:**
- The `Accept: application/vnd.github.raw` header returns the raw file content directly (not base64 encoded)
- Always use the `Authorization: Bearer ${GITHUB_TOKEN}` header for private repositories
- The API endpoint format is: `/repos/{owner}/{repo}/contents/{path}`
- For files in subdirectories, include the full path (e.g., `docs/architecture/README.md`)
- Do NOT use blob URLs (e.g., `https://github.com/.../blob/main/...`) as they return HTML, not file content
- If `GITHUB_TOKEN` is not set or invalid, you will receive a 401 Unauthorized or 404 Not Found error

### When to Use This Method

Use the GitHub REST API to fetch files when:
1. Reading referenced documentation from other private repositories
2. Retrieving templates or configuration files from organization repositories
3. Accessing files that are referenced in project-specific AGENTS.md files
4. Any automated operation that needs to read files from private GitHub repositories

**Do NOT** use this for public repositories where direct URL fetching would work without authentication.

---

## Additional Resources

### Documentation
- [Installation Guide](https://github.com/Ansible-SaaS/claude-session-manager/blob/main/docs/02-installation.md) - How to install and set up cs
- [Quick Start](https://github.com/Ansible-SaaS/claude-session-manager/blob/main/docs/03-quick-start.md) - Get started in 5 minutes
- [Session Management](https://github.com/Ansible-SaaS/claude-session-manager/blob/main/docs/04-session-management.md) - Working with sessions
- [JIRA Integration](https://github.com/Ansible-SaaS/claude-session-manager/blob/main/docs/05-jira-integration.md) - Optional JIRA features
- [Command Reference](https://github.com/Ansible-SaaS/claude-session-manager/blob/main/docs/07-commands.md) - Complete command list
- [Workflows](https://github.com/Ansible-SaaS/claude-session-manager/blob/main/docs/14-workflows.md) - Common usage patterns

### Project Info
- [GitHub Repository](https://github.com/Ansible-SaaS/claude-session-manager)
- [Full Documentation](https://github.com/Ansible-SaaS/claude-session-manager/tree/main/docs)

### Need Help?
Run `cs --help` or `cs <command> --help` for command-specific help.
