# Upload New Documents to Confluence

Process analyzed PDF documents, create Confluence pages with executive summaries, and organize files.

## Prerequisites

Run `/analyze-docs` first to generate `_analysis.json` and `_analysis.md` in the source folder. This command consumes that output.

## Source Folder
Ask the user for the source folder path if not already known from the `/analyze-docs` step.

## Steps

### 1. Load Analysis

Read `_analysis.json` from the source folder. If it doesn't exist, tell the user to run `/analyze-docs` first.

For each document in the analysis, you'll have: filename, title, authors, source organization, date, pages, type, tags, and executive summary.

### 2. Determine Target Space and Category

Fetch the current space and category structure from Confluence:

```bash
source .env  # or: export $(grep -v '^#' .env | xargs)

# List all spaces
curl -s -u "$CONFLUENCE_EMAIL:$CONFLUENCE_API_TOKEN" \
  "$CONFLUENCE_URL/rest/api/space" | jq '.results[] | {key, name}'
```

Refer to the Content Structure in CLAUDE.md for current spaces, categories, and page IDs.

For each document, use its tags and summary to determine placement. Present the proposed categorization to the user for confirmation:

| # | Document | Space | Category |
|---|----------|-------|----------|

If a document doesn't fit existing categories:
- Ask whether to create a new category under an existing space
- Or create a new space structure entirely
- Update CLAUDE.md with the new structure

### 3. Analyze Existing Page Structure for Consistency

Before creating pages, examine the target categories:

**Check 2-3 sibling pages** in each target category:
```bash
curl -s -u "$CONFLUENCE_EMAIL:$CONFLUENCE_API_TOKEN" \
  "$CONFLUENCE_URL/rest/api/content/{category_id}/child/page?expand=body.storage" | jq '.results[] | {title, id}'
```

Note: title naming conventions, heading hierarchy, macro placement, summary format and length.

**Check labels on sibling pages:**
```bash
curl -s -u "$CONFLUENCE_EMAIL:$CONFLUENCE_API_TOKEN" \
  "$CONFLUENCE_URL/rest/api/content/{sibling_id}/label"
```

### 4. Create Confluence Pages

For each document, create a page matching sibling conventions. Use the executive summary from `_analysis.json`.

Standard page format (adapt to match siblings):
```html
<p class="media-group"><ac:structured-macro ac:name="view-file" ac:schema-version="1">
  <ac:parameter ac:name="name"><ri:attachment ri:filename="document.pdf"/></ac:parameter>
</ac:structured-macro></p>
<h1>High-Level Summary</h1>
<p>Executive summary paragraphs...</p>
```

### 5. Upload PDF Attachments
```bash
curl -u "$CONFLUENCE_EMAIL:$CONFLUENCE_API_TOKEN" \
  -X POST \
  -H "X-Atlassian-Token: nocheck" \
  -F "file=@/path/to/document.pdf" \
  "$CONFLUENCE_URL/rest/api/content/{page_id}/child/attachment"
```

### 6. Add Labels

Map the document's tags from `_analysis.json` to Confluence labels. Check sibling page labels for conventions and add consistent labels:

```bash
curl -u "$CONFLUENCE_EMAIL:$CONFLUENCE_API_TOKEN" \
  -X POST \
  -H "Content-Type: application/json" \
  "$CONFLUENCE_URL/rest/api/content/{page_id}/label" \
  -d '[{"prefix":"global","name":"label-name"}]'
```

Always include the `document` label and the section label (e.g., `ai-risk`, `ai-implementation`).

### 7. Update Category Page (Required)

All category pages maintain a "Documents in this Section" table.

1. Fetch category page content and version number
2. Add new document(s) to the table **in alphabetical order**
3. Update with incremented version number

```bash
curl -s -u "$CONFLUENCE_EMAIL:$CONFLUENCE_API_TOKEN" \
  "$CONFLUENCE_URL/rest/api/content/{category_id}?expand=body.storage,version"
```

Row format:
```html
<tr><td><a href="/wiki/spaces/AI/pages/{page_id}">Document Title</a></td><td>Brief description</td><td>Mon YYYY</td></tr>
```

### 8. Update Parent Pages in Hierarchy (Required)

Traverse UP the page hierarchy and update document counts on all ancestor pages.

For each level (category -> section -> space home):
1. Fetch parent page content and version
2. Update document counts in tables, summaries, and site structure diagrams
3. Add new documents to "Recent Additions" or "What's New" sections
4. Update with incremented version number

### 9. Move Processed Files

Move PDFs from the source folder to their permanent local storage location.

Ask the user where to move the processed PDFs for permanent storage.

**Clean up:** Delete `_analysis.json`, `_analysis.md`, and any remaining text extraction files from the source folder.

### 10. Update Documentation

If new spaces, categories, or local folders were created:
- Update CLAUDE.md Content Structure with new page IDs and counts
- Document any new label conventions

### 11. Verify

- All pages display correctly and match sibling formatting
- PDF attachments are viewable via the view-file macro
- Labels are applied and consistent with category siblings
- Category page document tables include new documents in alphabetical order
- Date Added column is present and correctly populated
- All parent pages have updated document counts
- New documents appear in "Recent Additions" or "What's New" sections
