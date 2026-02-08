# Analyze PDF Documents

Analyze PDF documents in a folder: extract text, identify metadata, generate executive summaries, and suggest topic tags.

## Input

Folder path: `$ARGUMENTS`

If no argument is provided, default to `/Users/gquinn/documents/ai_and_reporting/new/`

## Steps

### 1. Load Taxonomy (if exists)

Check for a `_taxonomy.md` file in the source folder. If present, read it to understand:

- Existing tags and their groupings
- Source organizations previously encountered
- Any conventions or notes from prior runs

Use this as a reference for consistency -- not as a restriction. Prefer existing tags where the meaning genuinely overlaps, but propose new tags whenever the documents cover topics that aren't well captured by the current vocabulary. The taxonomy should evolve with the collection.

If no `_taxonomy.md` exists, that's fine -- infer everything from the document content.

### 2. Extract Text

**Do not read PDF files directly.** Use the extraction script with the source folder:

```bash
python3 /Users/gquinn/Documents/projects/confluence/extract_pdf_text.py --source "$FOLDER"
```

This creates text files in `<folder>_text/` with extracted content and metadata.

### 3. Read and Analyze Each Document

For each extracted text file, read enough content to identify:

- **Title**: The document's actual title (not the filename)
- **Author(s)**: Individual authors and/or publishing organization
- **Source Organization**: The institution behind the document
- **Date**: Publication date (month and year if available)
- **Page Count**: From the extraction metadata
- **Document Type**: paper, report, book, guide, framework, whitepaper, etc.

### 4. Generate Executive Summaries

For each document, write a concise executive summary:

- 3-4 substantive paragraphs
- High information density, no filler
- Paragraph 1: What the document is, who wrote it, and the core argument or finding
- Paragraph 2: Key methodology, evidence, or framework details
- Paragraph 3: Main conclusions, recommendations, or implications
- Preserve specific numbers, statistics, and named frameworks
- Attribute claims to their sources

### 5. Infer Topic Tags and Present for Confirmation

For each document, propose 5-10 topic tags. Use lowercase hyphenated format (e.g., `risk-management`, `supply-chain`, `machine-learning`).

**If a `_taxonomy.md` was loaded:** Use it as a reference, not a constraint. Prefer existing tags where they genuinely fit to maintain consistency, but freely propose new tags when the content covers topics not well served by the existing vocabulary. Mark new tags with `[NEW]` so they stand out during review.

**If no taxonomy exists:** Derive tags purely from document content. Tags should reflect the subject domain(s), key topics, methodologies, source organization, and publication year.

After analyzing all documents, **present the proposed tags and summaries to the user for review** before writing output files. Show a summary table:

| # | Title | Source | Date | Proposed Tags |
|---|-------|--------|------|---------------|

If proposing new tags not in the taxonomy, highlight them (e.g., mark with `[NEW]`).

Ask the user:
- Are these tags appropriate? Any to add, remove, or rename?
- Any documents needing different emphasis in their summary?

Incorporate feedback before proceeding.

### 6. Write Output Files

Write two files in the **source folder** (not the text extraction folder):

**`_analysis.md`** -- human-readable:

```markdown
# Document Analysis

Generated: [date]
Source folder: [path]
Documents analyzed: [count]

---

## 1. [Document Title]

- **File**: filename.pdf
- **Author(s)**: [names]
- **Source**: [organization]
- **Date**: [month year]
- **Pages**: [count]
- **Type**: [paper/report/guide/etc.]
- **Tags**: `tag-1`, `tag-2`, `tag-3`

### Summary

[3-4 paragraph executive summary]

---
```

**`_analysis.json`** -- structured for automation:

```json
{
  "generated": "YYYY-MM-DD",
  "source_folder": "/path/to/folder",
  "documents": [
    {
      "filename": "document.pdf",
      "title": "Document Title",
      "authors": ["Author Name"],
      "source_organization": "Organization",
      "date": "January 2026",
      "pages": 42,
      "type": "report",
      "tags": ["tag-1", "tag-2"],
      "summary": "Executive summary text..."
    }
  ]
}
```

### 7. Update Taxonomy (with approval)

If any new tags or source organizations were used that aren't in `_taxonomy.md`, propose additions:

- List new tags with their proposed grouping
- List new source organizations with type
- Show the proposed changes to the user and ask for approval

If approved, update `_taxonomy.md` with the new entries. If no `_taxonomy.md` existed, ask the user if they'd like to create one to preserve the taxonomy for future runs.

### 8. Clean Up

Delete the extracted text directory (`<folder>_text/`) after analysis is complete. The `_analysis.md`, `_analysis.json`, and `_taxonomy.md` remain in the source folder.
