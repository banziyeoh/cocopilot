---
name: jira-cli
description: Interact with Jira from the command line to create, list, view, edit, and transition issues, manage sprints and epics, and perform common Jira workflows. Use when the user asks about Jira tasks, tickets, issues, sprints, or needs to manage project work items.
license: MIT
compatibility: Requires jira-cli installed (https://github.com/ankitpokhrel/jira-cli) and configured with `jira init`. Requires JIRA_API_TOKEN environment variable.
metadata:
  author: Colby Timm
  version: "1.0"
---

# Jira CLI

Interact with Atlassian Jira from the command line using [jira-cli](https://github.com/ankitpokhrel/jira-cli).

## When to Use

- User asks to create, view, edit, or search Jira issues/tickets
- User needs to transition issues through workflow states (To Do → In Progress → Done)
- User wants to manage sprints, epics, or boards
- User needs to assign issues, add comments, or log work time
- User asks about their current tasks or sprint progress

## Prerequisites

### Installation

- **macOS**: `brew install ankitpokhrel/jira-cli/jira-cli`
- **Other platforms**: Download from [releases](https://github.com/ankitpokhrel/jira-cli/releases)
- **Docker**: `docker run -it --rm ghcr.io/ankitpokhrel/jira-cli:latest`

### Authentication Setup

#### Jira Cloud

1. Generate API token: https://id.atlassian.com/manage-profile/security/api-tokens
2. Export token: `export JIRA_API_TOKEN="your-api-token-here"`
3. Add to shell profile: `echo 'export JIRA_API_TOKEN="your-api-token"' >> ~/.zshrc`
4. Initialize: `jira init` → Select "Cloud" → Enter server URL and email

#### Jira On-Premise

1. For basic auth: Export password as `JIRA_API_TOKEN`
2. For PAT: Get token from Jira profile, export as `JIRA_API_TOKEN`, and set `JIRA_AUTH_TYPE=bearer`
3. For mTLS: Run `jira init` → Select "Local" → Select "mtls" auth → Provide certs
4. Initialize: `jira init` → Select "Local" → Enter server details

### Multiple Projects

```bash
# Use specific config file
jira issue list -c ./project-a-config.yaml

# Or set environment variable
JIRA_CONFIG_FILE=./project-a-config.yaml jira issue list
```

## Issue Commands

### List Issues

```bash
# List recent issues
jira issue list

# List my assigned issues
jira issue list -a$(jira me)

# List issues created in last 7 days
jira issue list --created -7d

# List issues by status
jira issue list -s"To Do"

# List high priority issues
jira issue list -yHigh

# List issues with multiple filters
jira issue list -a$(jira me) -s"To Do" -yHigh --created month -lbackend

# List in rank order (as shown in UI)
jira issue list --order-by rank --reverse

# List issues with raw JQL
jira issue list -q "summary ~ cli"

# List issues assigned to me and reported by another user
jira issue list -a$(jira me) -r"john.doe"

# List unassigned issues from this week
jira issue list -ax --created week

# List issues with resolution "Won't Do"
jira issue list --resolution "Won't Do"

# List issues created within an hour and updated in last 30 minutes
jira issue list --created -1h --updated -30m

# Output formats
jira issue list --plain           # Plain text (script-friendly)
jira issue list --raw             # Raw JSON
jira issue list --csv             # CSV format
jira issue list --plain --columns key,summary,status --no-headers
```

### Create Issues

```bash
# Interactive issue creation
jira issue create

# Create with all options specified
jira issue create -tBug -s"Login button not working" -b"Description here" -yHigh --no-input

# Create a story and attach to epic
jira issue create -tStory -s"User authentication" -PEPIC-42

# Create with labels and components
jira issue create -tTask -s"Update dependencies" -lmaintenance -l"tech-debt" -Cbackend

# Create and assign to self with fix version
jira issue create -tBug -s"Fix crash" -a$(jira me) --fix-version v2.0 --no-input

# Create with description from template (supports GitHub & Jira markdown)
jira issue create --template /path/to/template.tmpl

# Create with description from stdin
echo "Description from stdin" | jira issue create -s"Summary" -tTask

# Create with description from editor
jira issue create --template -

# Create with custom fields
jira issue create -tBug -s"Custom field example" --custom story-points=5
```

### View Issues

```bash
# View issue details
jira issue view ISSUE-123

# View with comments
jira issue view ISSUE-123 --comments 10

# View in plain text
jira issue view ISSUE-123 --plain

# Open issue in browser
jira open ISSUE-123
```

### Edit Issues

```bash
# Edit issue interactively
jira issue edit ISSUE-123

# Edit summary
jira issue edit ISSUE-123 -s"Updated summary"

# Edit description
jira issue edit ISSUE-123 -b"New description"

# Edit priority
jira issue edit ISSUE-123 -yHigh

# Add labels
jira issue edit ISSUE-123 -lnew-label

# Edit and skip interactive prompt
jira issue edit ISSUE-123 -s"New updated summary" --no-input

# Remove and add labels/components/fixVersions using minus (-)
# Remove label p2, add label p1
# Remove component FE, add component BE
# Remove fixVersion v1.0, add fixVersion v2.0
jira issue edit ISSUE-123 --label -p2 --label p1 --component -FE --component BE --fix-version -v1.0 --fix-version v2.0
```

### Transition Issues

```bash
# Interactive transition
jira issue move

# Move issue to a new status
jira issue move ISSUE-123 "In Progress"

# Move with comment
jira issue move ISSUE-123 "In Progress" --comment "Started working on it"

# Move to Done and set resolution
jira issue move ISSUE-123 "Done" -RFixed

# Move, set resolution, and assign to self
jira issue move ISSUE-123 "Done" -RFixed -a$(jira me)
```

**Note**: Press `m` in the interactive TUI to transition the selected issue.

### Assign Issues

```bash
# Interactive assignment
jira issue assign

# Assign to self
jira issue assign ISSUE-123 $(jira me)

# Assign to specific user
jira issue assign ISSUE-123 "Jon Doe"

# Will prompt for selection if keyword returns multiple users
jira issue assign ISSUE-123 suffix

# Assign to default assignee
jira issue assign ISSUE-123 default

# Unassign
jira issue assign ISSUE-123 x
```

### Comments

```bash
# Interactive comment
jira issue comment add

# Add a comment
jira issue comment add ISSUE-123 "This is my comment"

# Add an internal comment (visible only to users with permission)
jira issue comment add ISSUE-123 "Internal note" --internal

# Add comment from template (supports GitHub & Jira markdown)
jira issue comment add ISSUE-123 --template /path/to/template.tmpl

# Add comment from stdin
echo "Comment from stdin" | jira issue comment add ISSUE-123

# Add comment from editor
jira issue comment add ISSUE-123 --template -
```

**Note**: Positional argument takes precedence over `--template` flag if both are provided.

### Work Logging

```bash
# Log time
jira issue worklog add ISSUE-123 "2h 30m"

# Log time with comment
jira issue worklog add ISSUE-123 "1d 4h" --comment "Completed feature implementation" --no-input
```

### Link & Clone Issues

```bash
# Interactive link
jira issue link

# Link two issues
jira issue link ISSUE-123 ISSUE-456 Blocks

# Add remote web link to an issue
jira issue link remote ISSUE-123 https://example.com "Example text"

# Unlink issues using interactive prompt
jira issue unlink

# Unlink issues
jira issue unlink ISSUE-123 ISSUE-456

# Clone an issue
jira issue clone ISSUE-123

# Clone and modify fields
jira issue clone ISSUE-123 -s"Cloned: New summary" -yHigh -a$(jira me)

# Clone and replace text in summary and description (case-sensitive)
jira issue clone ISSUE-123 -H"find me:replace with me"

# Delete an issue
jira issue delete ISSUE-123

# Delete task with all subtasks
jira issue delete ISSUE-123 --cascade
```

## Epic Commands

```bash
# List epics (explorer view by default)
jira epic list

# List epics in table format
jira epic list --table

# List epics reported by me and are open
jira epic list -r$(jira me) -sOpen

# List issues in an epic
jira epic list EPIC-123

# List issues in epic with filters (all issue list flags work)
jira epic list EPIC-123 -ax -yHigh

# List epic issues in rank order (as shown in UI)
jira epic list EPIC-123 --order-by rank --reverse

# Create an epic interactively
jira epic create

# Create epic with fields
jira epic create -n"Q1 Features" -s"Epic for Q1 roadmap" -yHigh -lbug -lurgent -b"Epic description"

# Add issues to epic (up to 50 at once)
jira epic add EPIC-123 ISSUE-1 ISSUE-2

# Remove issues from epic (up to 50 at once)
jira epic remove ISSUE-1 ISSUE-2
```

## Sprint Commands

```bash
# List sprints (explorer view by default, shows 25 recent sprints)
jira sprint list

# List sprints in table format
jira sprint list --table

# List issues in current/active sprint
jira sprint list --current

# List my issues in current sprint
jira sprint list --current -a$(jira me)

# List issues in previous sprint
jira sprint list --prev

# List issues in next planned sprint
jira sprint list --next

# List sprints by state
jira sprint list --state future,active

# List issues in specific sprint (use sprint ID from list/table view)
jira sprint list SPRINT_ID

# List issues in sprint with filters (all issue list flags work)
jira sprint list SPRINT_ID -yHigh -a$(jira me)

# List sprint issues in rank order (as shown in UI)
jira sprint list SPRINT_ID --order-by rank --reverse

# Add issues to sprint (up to 50 at once)
jira sprint add SPRINT_ID ISSUE-123 ISSUE-456
```

## Project & Board Commands

```bash
# List projects
jira project list

# List boards
jira board list

# List releases/versions for default project
jira release list

# List releases for specific project by project ID
jira release list --project 1000

# List releases for specific project by project key
jira release list --project PROJ

# Open default project in browser
jira open

# Open specific issue in browser
jira open ISSUE-123
```

## UI Navigation (Interactive Mode)

When viewing issues in the interactive TUI:

| Key                | Action                         |
| ------------------ | ------------------------------ |
| `j, k, h, l, ↑, ↓` | Navigate list                  |
| `g`                | Jump to top                    |
| `G`                | Jump to bottom                 |
| `CTRL + f`         | Scroll page down               |
| `CTRL + b`         | Scroll page up                 |
| `v`                | View selected issue details    |
| `m`                | Transition/move selected issue |
| `ENTER`            | Open issue in browser          |
| `c`                | Copy issue URL to clipboard    |
| `CTRL + k`         | Copy issue key to clipboard    |
| `w` or `TAB`       | Toggle focus (sidebar/content) |
| `CTRL + r` or `F5` | Refresh issues list            |
| `?`                | Open help window               |
| `q, ESC, CTRL + c` | Quit                           |

## Utility Commands

```bash
# Get current username
jira me

# Show help
jira --help
jira issue --help

# Shell completion setup
jira completion bash  # or zsh, fish, powershell
```

## Common Flags

| Flag               | Description                                        |
| ------------------ | -------------------------------------------------- |
| `--plain`          | Plain text output (no interactive UI)              |
| `--table`          | Table view (for epics/sprints)                     |
| `--raw`            | Raw JSON output                                    |
| `--csv`            | CSV output                                         |
| `--no-input`       | Skip interactive prompts                           |
| `--no-headers`     | Omit headers in output                             |
| `--columns`        | Specify columns to display                         |
| `-c, --config`     | Path to specific config file                       |
| `-t, --type`       | Issue type (Bug, Story, Task, Epic)                |
| `-s, --summary`    | Issue summary/title                                |
| `-b, --body`       | Issue description                                  |
| `-y, --priority`   | Priority (Highest, High, Medium, Low, Lowest)      |
| `-l, --label`      | Labels (repeatable, use `-` prefix to remove)      |
| `-a, --assignee`   | Assignee username (`x` to unassign)                |
| `-r, --reporter`   | Reporter username                                  |
| `-C, --component`  | Component name (repeatable, use `-` to remove)     |
| `-P, --parent`     | Parent issue/epic key                              |
| `-R, --resolution` | Resolution (Fixed, Won't Fix, Duplicate, etc.)     |
| `-q, --jql`        | Raw JQL query                                      |
| `-n, --name`       | Epic name (for epic creation)                      |
| `--created`        | Filter by creation date (-7d, week, month, -1h)    |
| `--updated`        | Filter by update date (-30m, -1h, -7d)             |
| `--order-by`       | Sort field (default: created)                      |
| `--reverse`        | Reverse sort order                                 |
| `--fix-version`    | Fix version (repeatable, use `-` to remove)        |
| `--template`       | Load content from template file (or `-` for stdin) |
| `--internal`       | Mark comment as internal (visible to admins only)  |
| `--custom`         | Set custom fields (e.g., `--custom field=value`)   |
| `--comment`        | Add comment when transitioning                     |
| `--comments`       | Number of comments to show when viewing issue      |
| `--cascade`        | Delete with subtasks                               |
| `--current`        | Current/active sprint                              |
| `--prev`           | Previous sprint                                    |
| `--next`           | Next sprint                                        |
| `--state`          | Sprint state filter (future, active, closed)       |

## Common Workflows

### Start Working on an Issue

```bash
# Assign to self and move to In Progress
jira issue assign ISSUE-123 $(jira me)
jira issue move ISSUE-123 "In Progress"
```

### Complete an Issue

```bash
# Log work, add comment, and close with resolution
jira issue worklog add ISSUE-123 "4h" --comment "Feature implementation complete" --no-input
jira issue move ISSUE-123 "Done" --comment "Completed and tested" -RFixed
```

### Daily Standup Review

```bash
# View my current sprint tasks
jira sprint list --current -a$(jira me)

# View what I worked on yesterday (updated in last 24h)
jira issue list -a$(jira me) --updated -24h
```

### Create and Track a Bug

```bash
# Create bug with all details
BUG_KEY=$(jira issue create -tBug -s"App crashes on login" -yHigh -lbug -b"Steps to reproduce..." --no-input --plain | grep -o '[A-Z]\+-[0-9]\+')

# Assign to self and start work
jira issue assign $BUG_KEY $(jira me)
jira issue move $BUG_KEY "In Progress"
```

### Weekly Report

```bash
# Issues I completed this week
jira issue list -a$(jira me) -s"Done" --updated week

# Issues I created this week
jira issue list -r$(jira me) --created week
```

### Find My First Bug Fixed

```bash
# First bug ever fixed on current board
jira issue list -a$(jira me) -tBug -sDone --order-by created --reverse | head -1
```

## JQL Query Examples

Use `-q` or `--jql` flag for complex queries:

```bash
# Issues I am watching
jira issue list -q "watcher = currentUser()"

# Issues assigned to me and reported by another user
jira issue list -q "assignee = currentUser() AND reporter != currentUser()"

# Open high priority issues assigned to me
jira issue list -q "assignee = currentUser() AND priority = High AND status != Done"

# Issues with no assignee created this week
jira issue list -q "assignee is EMPTY AND created >= -1w"

# Issues with resolution "Won't Do"
jira issue list -q "resolution = 'Won't Do'"

# Issues not done, created before 6 months ago, assigned to someone
jira issue list -q "status != Done AND created <= -26w AND assignee is not EMPTY"

# Issues created within an hour and updated in last 30 minutes
jira issue list -q "created >= -1h AND updated >= -30m"

# High priority in progress issues from this month with labels
jira issue list -q "priority = High AND status = 'In Progress' AND created >= startOfMonth() AND labels in (backend, urgent)"

# Issues I reported today
jira issue list -q "reporter = currentUser() AND created >= startOfDay()"

# Am I watching any tickets in project XYZ?
jira issue list -q "project = XYZ AND watcher = currentUser()"

# First issue I ever reported on current board
jira issue list -q "reporter = currentUser() ORDER BY created ASC" --plain | head -1
```

## Output Examples

| Command                   | Use Case               |
| ------------------------- | ---------------------- |
| `jira issue list --plain` | Script-friendly output |
| `jira issue list --raw`   | JSON for parsing       |
| `jira issue list --csv`   | Export to spreadsheet  |

## Limitations & Notes

- Requires prior `jira init` configuration with valid API token
- Some features may vary between Jira Cloud and Server/Data Center
- Complex custom fields may require `--custom` flag with field IDs (see [discussion](https://github.com/ankitpokhrel/jira-cli/discussions/346))
- Rate limits apply based on Jira instance configuration
- On-premise non-English installations may need manual config for `epic.name`, `epic.link`, and `issue.types.*.handle` fields
- Not all [Atlassian document nodes](https://developer.atlassian.com/cloud/jira/platform/apis/document/structure/#nodes) translate perfectly (may cause formatting issues)
- Interactive UI requires terminal with proper TTY support
- Copy to clipboard requires `xclip` or `xsel` on Linux
- Custom pager can be configured (default is `less`) - see [discussion](https://github.com/ankitpokhrel/jira-cli/discussions/569)
- View command shows latest comment (may not be newest if >5000 comments exist)
- Sprint list shows 25 most recent sprints only

## Resources

- [Official Repository](https://github.com/ankitpokhrel/jira-cli)
- [Installation Guide](https://github.com/ankitpokhrel/jira-cli/wiki/Installation)
- [FAQs](https://github.com/ankitpokhrel/jira-cli/discussions/categories/faqs)
- [Feature Requests](https://github.com/ankitpokhrel/jira-cli/discussions/categories/ideas)
- [Getting Started Blog Post](https://www.mslinn.com/blog/2022/08/12/jiracli.html)
