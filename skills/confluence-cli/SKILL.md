---
name: confluence-cli
description: Interact with Confluence from the command line to create, read, update, and delete pages, manage comments and attachments, export content, and perform common Confluence workflows. Use when the user asks about Confluence pages, documentation, wiki content, or needs to manage Confluence spaces.
license: MIT
compatibility: Requires confluence-cli installed via npm (https://github.com/pchuri/confluence-cli) and configured with Confluence API token.
metadata:
  author: Colby Timm
  version: "1.0"
---

# Confluence CLI

Interact with Atlassian Confluence from the command line using [confluence-cli](https://github.com/pchuri/confluence-cli).

## When to Use

- User asks to read, create, update, or delete Confluence pages
- User needs to search for documentation or wiki content
- User wants to manage page attachments or comments
- User needs to export pages with attachments
- User asks to list spaces or find pages by title
- User needs to copy page trees or manage page hierarchy
- User wants to edit page content from the terminal

## Prerequisites

- Requires disabling the SSL checking before use with self-signed certificates: `export NODE_TLS_REJECT_UNAUTHORIZED=0`

### Installation

```bash
# Install globally
npm install -g confluence-cli

# Or run directly with npx
npx confluence-cli
```

### Authentication Setup

#### Option 1: Interactive Setup

```bash
confluence init
```

The wizard helps you choose the right API endpoint and authentication method. It recommends `/wiki/rest/api` for Atlassian Cloud domains (e.g., `*.atlassian.net`) and `/rest/api` for self-hosted/Data Center instances, then prompts for Basic (email + token) or Bearer authentication.

#### Option 2: Non-Interactive Setup (CLI Flags)

Perfect for CI/CD pipelines, Docker builds, and AI coding agents.

```bash
# Complete non-interactive mode (all required fields provided)
confluence init \
  --domain "company.atlassian.net" \
  --api-path "/wiki/rest/api" \
  --auth-type "basic" \
  --email "user@example.com" \
  --token "your-api-token"

# Hybrid mode (some fields provided, rest via prompts)
confluence init --domain "company.atlassian.net" --token "your-api-token"

# Email indicates basic auth, will prompt for domain and token
confluence init --email "user@example.com" --token "your-api-token"
```

**Available flags:**

- `-d, --domain <domain>` - Confluence domain (e.g., `company.atlassian.net`)
- `-p, --api-path <path>` - REST API path (e.g., `/wiki/rest/api`)
- `-a, --auth-type <type>` - Authentication type: `basic` or `bearer`
- `-e, --email <email>` - Email for basic authentication
- `-t, --token <token>` - API token

⚠️ **Security note:** While flags work, storing tokens in shell history is risky. Prefer environment variables for production environments.

#### Option 3: Environment Variables

```bash
export CONFLUENCE_DOMAIN="your-domain.atlassian.net"
export CONFLUENCE_API_TOKEN="your-api-token"
export CONFLUENCE_EMAIL="your.email@example.com"  # Required for basic auth
export CONFLUENCE_API_PATH="/wiki/rest/api"       # Cloud default; use /rest/api for Server/DC
export CONFLUENCE_AUTH_TYPE="basic"               # Optional: 'basic' or 'bearer'
```

`CONFLUENCE_API_PATH` defaults to `/wiki/rest/api` for Atlassian Cloud domains and `/rest/api` otherwise. `CONFLUENCE_AUTH_TYPE` defaults to `basic` when an email is present and falls back to `bearer` otherwise.

### Getting Your API Token

1. Go to [Atlassian Account Settings](https://id.atlassian.com/manage-profile/security/api-tokens)
2. Click "Create API token"
3. Give it a label (e.g., "confluence-cli")
4. Copy the generated token

## Page Commands

### Read Pages

```bash
# Read by page ID
confluence read 123456789

# Read in markdown format
confluence read 123456789 --format markdown

# Read in HTML format
confluence read 123456789 --format html

# Read by URL (must contain pageId parameter)
confluence read "https://your-domain.atlassian.net/wiki/viewpage.action?pageId=123456789"

# Get page information (metadata)
confluence info 123456789
```

### Create Pages

```bash
# Create with inline content (markdown format)
confluence create "My New Page" SPACEKEY --content "**Hello** World!" --format markdown

# Create from a file
confluence create "Documentation" SPACEKEY --file ./content.md --format markdown

# Create from a file in HTML format
confluence create "API Docs" SPACEKEY --file ./api-docs.html --format html

# Create from a file in storage format (Confluence native format)
confluence create "Technical Specs" SPACEKEY --file ./specs.xml --format storage
```

### Create Child Pages

```bash
# Create child page with inline content
confluence create-child "Meeting Notes" 123456789 --content "This is a child page"

# Create child page from a file
confluence create-child "Tech Specs" 123456789 --file ./specs.md --format markdown

# Create child page in HTML format
confluence create-child "Design Document" 123456789 --file ./design.html --format html
```

### Update Pages

```bash
# Update title only
confluence update 123456789 --title "Updated Page Title"

# Update content only from a string
confluence update 123456789 --content "Updated page content."

# Update content from a file
confluence update 123456789 --file ./updated-content.md --format markdown

# Update both title and content
confluence update 123456789 --title "New Title" --content "And new content"

# Update with HTML format
confluence update 123456789 --file ./content.html --format html
```

### Delete Pages

```bash
# Delete by page ID (prompts for confirmation)
confluence delete 123456789

# Delete by URL
confluence delete "https://your-domain.atlassian.net/wiki/viewpage.action?pageId=123456789"

# Skip confirmation (useful for scripts)
confluence delete 123456789 --yes
```

## Search & Navigation Commands

### Search Pages

```bash
# Basic search
confluence search "search term"

# Limit results
confluence search "API documentation" --limit 5

# Search for multiple terms
confluence search "deployment guide production"
```

### Find Pages by Title

```bash
# Find page by title
confluence find "Project Documentation"

# Find page by title in a specific space
confluence find "Project Documentation" --space MYTEAM
```

### List Child Pages

```bash
# List direct child pages
confluence children 123456789

# List all descendants recursively
confluence children 123456789 --recursive

# Display as tree structure
confluence children 123456789 --recursive --format tree

# Show page IDs and URLs
confluence children 123456789 --show-id --show-url

# Limit recursion depth
confluence children 123456789 --recursive --max-depth 3

# Output as JSON for scripting
confluence children 123456789 --recursive --format json > children.json
```

### List Spaces

```bash
# List all available spaces
confluence spaces
```

## Comments Commands

### List Comments

```bash
# List all comments (footer + inline)
confluence comments 123456789

# List footer comments only
confluence comments 123456789 --location footer

# List inline comments as markdown
confluence comments 123456789 --location inline --format markdown

# List with pagination
confluence comments 123456789 --limit 10 --start 0

# List resolved comments
confluence comments 123456789 --location resolved

# List root comments only (no replies)
confluence comments 123456789 --depth root

# List all comments including replies
confluence comments 123456789 --depth all

# Output as JSON
confluence comments 123456789 --format json
```

### Create Comments

```bash
# Create a footer comment
confluence comment 123456789 --content "Looks good to me!"

# Create a footer comment from a file
confluence comment 123456789 --file ./review-notes.md --format markdown

# Reply to a comment
confluence comment 123456789 --parent 998877 --content "Agree with this"

# Create an inline comment (requires highlight metadata)
confluence comment 123456789 \
  --location inline \
  --content "Consider renaming this" \
  --inline-selection "foo" \
  --inline-original-selection "foo"

# Create comment in HTML format
confluence comment 123456789 --content "<p>HTML comment</p>" --format html
```

**Note:** Creating inline comments on Confluence Cloud requires editor-generated highlight metadata. The public REST API does not provide these fields, so inline creation may fail with a 400 error unless you supply the full `--inline-properties` payload. Footer comments and replies are fully supported.

### Delete Comments

```bash
# Delete a comment (prompts for confirmation)
confluence comment-delete 998877

# Delete without confirmation
confluence comment-delete 998877 --yes
```

## Attachments Commands

```bash
# List all attachments on a page
confluence attachments 123456789

# Filter by filename pattern and limit results
confluence attachments 123456789 --pattern "*.png" --limit 5

# Download matching attachments to a directory
confluence attachments 123456789 --pattern "*.pdf" --download --dest ./downloads

# Download all attachments
confluence attachments 123456789 --download --dest ./all-attachments
```

## Copy & Export Commands

### Copy Page Tree

```bash
# Copy a page and all its children to a new location
confluence copy-tree 123456789 987654321 "Project Docs (Copy)"

# Copy with maximum depth limit (only 3 levels deep)
confluence copy-tree 123456789 987654321 --max-depth 3

# Exclude pages by title (supports wildcards * and ?; case-insensitive)
confluence copy-tree 123456789 987654321 --exclude "temp*,test*,*draft*"

# Control pacing and naming
confluence copy-tree 123456789 987654321 --delay-ms 150 --copy-suffix " (Backup)"

# Dry run (preview only)
confluence copy-tree 123456789 987654321 --dry-run

# Quiet mode (suppress progress output)
confluence copy-tree 123456789 987654321 --quiet

# Fail on first error
confluence copy-tree 123456789 987654321 --fail-on-error
```

**Notes:**

- Preserves the original parent-child hierarchy when copying
- Continues on errors: failed pages are logged and the copy proceeds
- Exclude patterns use simple globbing: `*` matches any sequence, `?` matches any single character
- Large trees may take time; the CLI applies a small delay between sibling page creations to avoid rate limits
- Root title suffix defaults to ` (Copy)`; override with `--copy-suffix`
- Child pages keep their original titles
- Use `--fail-on-error` to exit non-zero if any page fails to copy

### Export Pages

```bash
# Export page content (markdown by default) and all attachments
confluence export 123456789 --dest ./exports

# Export in HTML format
confluence export 123456789 --format html --dest ./exports

# Custom content format/filename and attachment filtering
confluence export 123456789 --format html --file content.html --pattern "*.png"

# Skip attachments if you only need the content file
confluence export 123456789 --skip-attachments

# Export only referenced attachments
confluence export 123456789 --referenced-only

# Export to specific directory with custom attachment folder
confluence export 123456789 --dest ./my-export --attachments-dir files
```

## Edit Workflow

The `edit` and `update` commands work together to create a seamless editing workflow:

```bash
# 1. Export page content to a file (in Confluence storage format)
confluence edit 123456789 --output ./page-to-edit.xml

# 2. Edit the file with your preferred editor
vim ./page-to-edit.xml
# or
code ./page-to-edit.xml

# 3. Update the page with your changes
confluence update 123456789 --file ./page-to-edit.xml --format storage
```

## Utility Commands

```bash
# View usage statistics
confluence stats

# Show help
confluence --help

# Show help for specific command
confluence read --help
```

## Common Flags

| Flag                          | Description                                              |
| ----------------------------- | -------------------------------------------------------- |
| `--format`                    | Content format: `html`, `text`, `markdown`, or `storage` |
| `--file`                      | Path to file for content input/output                    |
| `--content`                   | Inline content string                                    |
| `--title`                     | Page title                                               |
| `--dest`                      | Destination directory for exports/downloads              |
| `--pattern`                   | Glob pattern for filtering (e.g., `*.png`)               |
| `--limit`                     | Maximum number of results                                |
| `--start`                     | Starting index for pagination                            |
| `--space`                     | Space key for filtering                                  |
| `--recursive`                 | Process recursively (for children command)               |
| `--max-depth`                 | Maximum recursion depth                                  |
| `--show-id`                   | Show page IDs in output                                  |
| `--show-url`                  | Show page URLs in output                                 |
| `--download`                  | Download attachments                                     |
| `--location`                  | Comment location: `footer`, `inline`, or `resolved`      |
| `--parent`                    | Parent comment ID for replies                            |
| `--inline-selection`          | Text selection for inline comments                       |
| `--inline-original-selection` | Original text for inline comments                        |
| `--inline-marker-ref`         | Marker reference for inline comments                     |
| `--inline-properties`         | Full inline comment properties JSON                      |
| `--depth`                     | Comment depth: `root` (no replies) or `all`              |
| `--all`                       | Include all results                                      |
| `--yes`, `-y`                 | Skip confirmation prompts                                |
| `--output`, `-o`              | Output file path                                         |
| `--skip-attachments`          | Skip downloading attachments                             |
| `--referenced-only`           | Export only attachments referenced in page content       |
| `--attachments-dir`           | Custom name for attachments directory                    |
| `--exclude`                   | Comma-separated patterns to exclude (supports wildcards) |
| `--delay-ms`                  | Delay in milliseconds between operations                 |
| `--copy-suffix`               | Suffix to add to copied page titles                      |
| `--dry-run`                   | Preview without making changes                           |
| `--quiet`                     | Suppress progress output                                 |
| `--fail-on-error`             | Exit with error code on first failure                    |
| `-d, --domain`                | Confluence domain (init command)                         |
| `-p, --api-path`              | REST API path (init command)                             |
| `-a, --auth-type`             | Authentication type: `basic` or `bearer` (init command)  |
| `-e, --email`                 | Email for basic authentication (init command)            |
| `-t, --token`                 | API token (init command)                                 |

## Common Workflows

### Create and Publish Documentation

```bash
# Create a main documentation page
PAGE_ID=$(confluence create "Product Documentation" PRODSPACE \
  --file ./docs/overview.md \
  --format markdown | grep -o '[0-9]\+')

# Create child pages for each section
confluence create-child "Getting Started" $PAGE_ID --file ./docs/getting-started.md --format markdown
confluence create-child "API Reference" $PAGE_ID --file ./docs/api-reference.md --format markdown
confluence create-child "Troubleshooting" $PAGE_ID --file ./docs/troubleshooting.md --format markdown
```

### Backup Space Content

```bash
# Find all pages in a space
confluence search "space = MYSPACE" --limit 100 > page-list.txt

# Export each page with attachments
while read page_id; do
  confluence export $page_id --dest ./backups/$page_id
done < page-ids.txt
```

### Documentation Review Workflow

```bash
# Search for pages needing review
confluence search "label = needs-review"

# Add review comments
confluence comment 123456789 --content "Reviewed and approved for publication"

# Update page after revision
confluence update 123456789 --file ./revised-content.md --format markdown
```

### Clone Documentation to New Space

```bash
# Find the root page of documentation
ROOT_PAGE=$(confluence find "Product Docs" --space OLDSPACE | grep -o '[0-9]\+')

# Find target space root
TARGET_ROOT=$(confluence find "Home" --space NEWSPACE | grep -o '[0-9]\+')

# Copy entire tree
confluence copy-tree $ROOT_PAGE $TARGET_ROOT "Product Docs (New)"
```

### Daily Documentation Update

```bash
# Export for editing
confluence edit 123456789 --output ./daily-update.xml

# Make changes in your editor
code ./daily-update.xml

# Update the page
confluence update 123456789 --file ./daily-update.xml --format storage

# Add a comment about the update
confluence comment 123456789 --content "Updated with latest product changes"
```

### Batch Download Attachments

```bash
# Find pages with attachments
confluence search "hasAttachments"

# Download all PDF attachments from a page
confluence attachments 123456789 --pattern "*.pdf" --download --dest ./pdfs

# Download all images
confluence attachments 123456789 --pattern "*.{png,jpg,jpeg}" --download --dest ./images
```

### Export Team Documentation

```bash
# Export main pages with all attachments
confluence export 123456789 --dest ./team-docs/onboarding
confluence export 987654321 --dest ./team-docs/processes
confluence export 456789123 --dest ./team-docs/standards

# Create archive
tar -czf team-docs-$(date +%Y%m%d).tar.gz ./team-docs/
```

## Output Examples

| Command                                   | Use Case                   |
| ----------------------------------------- | -------------------------- |
| `confluence read 123456789 --format text` | Plain text output          |
| `confluence children 123 --format json`   | JSON for parsing/scripting |
| `confluence children 123 --format tree`   | Visual tree structure      |
| `confluence export 123 --format markdown` | Export to Markdown         |

## Script Examples

### Find and Update Multiple Pages

```bash
#!/bin/bash
# Find all pages with a specific label and update them

PAGES=$(confluence search "label = outdated" --limit 50)

for page_id in $PAGES; do
  echo "Updating page $page_id"
  confluence comment $page_id --content "This page has been flagged for review"
done
```

### Automated Documentation Sync

```bash
#!/bin/bash
# Sync local documentation to Confluence

SPACE="DOCS"
ROOT_PAGE="Project Documentation"

# Find or create root page
ROOT_ID=$(confluence find "$ROOT_PAGE" --space $SPACE)

if [ -z "$ROOT_ID" ]; then
  echo "Creating root page"
  ROOT_ID=$(confluence create "$ROOT_PAGE" $SPACE --content "Main documentation" | grep -o '[0-9]\+')
fi

# Update pages from local files
for file in ./docs/*.md; do
  title=$(basename "$file" .md)

  # Try to find existing page
  page_id=$(confluence find "$title" --space $SPACE)

  if [ -z "$page_id" ]; then
    # Create new page
    echo "Creating page: $title"
    confluence create-child "$title" $ROOT_ID --file "$file" --format markdown
  else
    # Update existing page
    echo "Updating page: $title"
    confluence update $page_id --file "$file" --format markdown
  fi
done

echo "Documentation sync complete"
```

### Generate Space Report

```bash
#!/bin/bash
# Generate report of space content

SPACE="MYSPACE"
OUTPUT="space-report.md"

echo "# Space Report: $SPACE" > $OUTPUT
echo "Generated: $(date)" >> $OUTPUT
echo "" >> $OUTPUT

# List all spaces
echo "## Spaces" >> $OUTPUT
confluence spaces >> $OUTPUT

# Search for pages
echo "" >> $OUTPUT
echo "## Recent Pages" >> $OUTPUT
confluence search "space = $SPACE" --limit 10 >> $OUTPUT

echo "Report generated: $OUTPUT"
```

## Limitations & Notes

- Requires disabling the SSL checking before use with self-signed certificates: `export NODE_TLS_REJECT_UNAUTHORIZED=0`
- Some features may vary between Confluence Cloud and Server/Data Center
- Rate limits apply based on Confluence instance configuration
- Inline comment creation on Confluence Cloud requires editor-generated metadata not available through public API
- Large page trees may take time to copy/export; consider using `--delay-ms` to avoid rate limiting
- The `copy-tree` command continues on errors by default; use `--fail-on-error` to stop on first failure
- Export command may not perfectly preserve all Confluence storage format features when converting to markdown/HTML
- Analytics can be disabled by setting `CONFLUENCE_CLI_ANALYTICS=false`
- API path defaults to `/wiki/rest/api` for Cloud and `/rest/api` for self-hosted instances
- Authentication defaults to `basic` when email is provided, otherwise `bearer`

## Resources

- [Official Repository](https://github.com/pchuri/confluence-cli)
- [NPM Package](https://www.npmjs.com/package/confluence-cli)
- [Contributing Guide](https://github.com/pchuri/confluence-cli/blob/main/CONTRIBUTING.md)
- [Issues & Bug Reports](https://github.com/pchuri/confluence-cli/issues)
- [Feature Requests](https://github.com/pchuri/confluence-cli/issues/new?template=feature_request.md)
- [Discussions](https://github.com/pchuri/confluence-cli/discussions)
- [Atlassian API Documentation](https://developer.atlassian.com/cloud/confluence/rest/v1/intro/)
- [Confluence Storage Format](https://confluence.atlassian.com/doc/confluence-storage-format-790796544.html)
