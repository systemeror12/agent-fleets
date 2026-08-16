---
name: postplan-read
description: Use when the user provides a Postplan link or asks to read, inspect, or summarize a Postplan draft.
---

# Postplan Read

Read the published Postplan draft and answer from its actual HTML content.

## Workflow

1. Identify the source. Use the Postplan URL provided by the user. If the user gives only a draft ID, fetch `https://postplan.dev/d/<draft-id>/raw`.

2. Fetch the document with the available web, browser, or HTTP tools. Prefer the raw URL when exact HTML or source structure matters. Confirm that the response loaded successfully before interpreting it.

3. Read the document semantically. Preserve the document's headings, facts, dates, metrics, links, decisions, risks, and next steps. Answer the user's specific question first, then summarize the relevant context.

4. Distinguish source content from interpretation. Quote only the short passages needed to support an answer, label inferences, and do not invent missing details.

5. For a review request, inspect structure, clarity, accessibility, links, and visible presentation. Report findings without changing or republishing the draft unless the user explicitly asks for that work.

6. If the URL cannot be fetched or the draft is unavailable, report the exact access problem and request a working URL or draft ID. Do not infer the document's contents.

Complete the task when the response is grounded in the fetched Postplan draft and directly answers the user's request.
