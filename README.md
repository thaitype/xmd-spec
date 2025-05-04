# XMD Spec: Extended Markdown Format

**Spec Version**: xmd-spec.thaitype.dev/v1
**Status**: Draft (2025-05-04)
**Primary Use Case**: Structured Documents with Block-level Metadata

## Overview

XMD (Extended Markdown) is a Markdown-compatible document format that adds structured, machine-readable metadata to individual content blocks. It is designed to be easy to read, easy to version, and simple to extend — without sacrificing compatibility with existing Markdown tools.

Each block in an XMD file can include inline metadata to describe its identity, language, change status, or other domain-specific properties. This makes XMD ideal for:

* Selective processing (e.g. embedding only changed content for LLMs)
* Translation workflows (e.g. aligning translated blocks with originals)
* Document synchronization (e.g. syncing only updated paragraphs from upstream sources)

Because XMD is based on plain Markdown with structured comments, you can preview or diff it just like any `.md` file — while automated systems can extract block-level intelligence from it.

In this spec, you'll learn how XMD is structured, how to annotate content with metadata, and how to use it to power partial processing pipelines and document-aware systems.

## Motivation

Modern workflows often rely on Markdown for human-readable documents, but Markdown alone lacks structured metadata needed for automation, change tracking, or selective processing. XMD was created to fill this gap — enabling documents to remain Markdown-compatible while embedding machine-readable metadata per block.

This spec emerged from a real-world challenge: converting Notion documents to Markdown and syncing them in batch jobs (e.g., monthly). Without inline metadata, it's impossible to know which parts of which documents have changed — requiring reprocessing entire files unnecessarily. This is inefficient, especially for older documents that rarely change.

XMD allows each paragraph or block to carry metadata like a hash, language, or last-updated info. This supports workflows such as:

* **Incremental downloading**: Only fetch updated blocks from Notion instead of entire documents
* **Partial LLM embedding**: Avoid recomputing embeddings for unchanged content
* **Selective translation**: Translate or re-translate only blocks that have changed

In short, XMD supports efficient, incremental, and metadata-aware document workflows — while preserving compatibility with traditional Markdown tools.


## File Structure

An `.x.md` file consists of:

1. **YAML Frontmatter** with global metadata
2. **Markdown content** interleaved with `@meta` comments in JSON format

## 1. YAML Frontmatter

```yaml
---
apiVersion: xmd-spec.thaitype.dev/v1
metadata:
  documentId: guide-001
  title: "LLM Prompting Guide"
  lang: en
  createdAt: 2025-05-04T10:30:00Z
  tags: [llm, prompting]
---
```

### Required Fields

* `apiVersion`: The spec version (must be `xmd-spec.thaitype.dev/v1`)
* `metadata`: Document-level info

## 2. Paragraph Metadata Blocks (`@meta`)

Each content block can be preceded by a single-line HTML comment with metadata:

```markdown
<!-- @meta { "id": "p1", "hash": "abc123", "lang": "en" } -->
This is the first paragraph that will be embedded.
```

### Notes:

* The comment must immediately precede the paragraph (no empty line)
* JSON inside `@meta` is free-form, but `id` and `hash` are strongly recommended
* Parsers should treat the paragraph and its `@meta` comment as a single block

## Example JSON Output Format (After Parsing)

```json
{
  "apiVersion": "xmd-spec.thaitype.dev/v1",
  "metadata": {
    "documentId": "guide-001",
    "title": "LLM Prompting Guide",
    "lang": "en",
    "createdAt": "2025-05-04T10:30:00Z"
  },
  "blocks": [
    {
      "id": "p1",
      "hash": "abc123",
      "lang": "en",
      "content": "Prompting is the art of designing clear instructions for LLMs.",
      "embedding": [0.11, -0.32, 0.91]
    },
    {
      "id": "p1-th",
      "ref": "p1",
      "hash": "xyz789",
      "lang": "th",
      "status": "translated",
      "content": "การตั้งคำถามคือศิลปะในการสื่อสารกับโมเดลภาษา",
      "embedding": null
    }
  ]
}
```

## Design Philosophy

* **Markdown-Compatible**: All `.x.md` files are valid Markdown
* **Flexible Metadata**: Use-case driven per-block metadata
* **LLM-Friendly**: Enables fine-grained control of embedding and translation pipelines
* **Versioned Spec**: Enables backward/forward compatibility

## Suggested Workflow for Partial Embedding

1. Parse `.x.md` to extract blocks and metadata
2. Compare `hash` values with stored versions
3. Embed only changed or missing blocks
4. Update `embedding` + `lastEmbeddedAt` (outside XMD or in separate index)

## Application Design Challenges

While XMD provides a minimal and flexible format for structured Markdown, implementers should be aware of the following practical challenges when applying it to real-world systems:

### Usability Considerations

* Users authoring `.x.md` manually should use editor support or CLI tooling to generate consistent `id` and `hash` fields
* Block-level metadata depends on proximity — misplaced spacing can break association
* Systems should validate uniqueness of `id` fields within and across documents to avoid conflicts

### Scaling with Large Documents

* Processing thousands of blocks efficiently may require indexing and caching layers to avoid re-hashing or re-embedding all content
* External tracking of moved or renamed blocks is necessary, as XMD does not encode cross-file lineage
* Parallel processing requires clear isolation of blocks and careful handling of file IO

### Structural Limitations

* XMD blocks are flat; systems needing tree-like document structures must build additional layers (e.g. parentId, outline maps)
* Inline-level metadata is currently unsupported, limiting fine-grained annotation or phrase-level processing

These tradeoffs are intentional: XMD prioritizes compatibility and simplicity. Systems using XMD should layer their own conventions, validation, and tooling to meet specific operational needs.

## Gap of Improvements 

In the future, we may consider extending XMD to support:

* Inline metadata for spans or tokens
* Structured block types (`type: heading`, `list`, etc.)
* Parent-child structure (tree format)
* External index files for workspace-wide deduplication

## License

This specification is open and community-driven under CC BY 4.0. Contributions and feedback are welcome via GitHub issues or pull requests.
