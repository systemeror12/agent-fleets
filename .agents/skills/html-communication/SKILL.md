---
name: html-communication
description: Use when the user asks for a standalone HTML writeup for work outside a codebase.
---

# HTML Communication

Create a standalone HTML work writeup and publish it through Postplan. Treat every output as a dense, readable document.

## Content specification

- Write like a specification, not a landing page.
- Make the document dense, factual, and scannable.
- Use direct language and a clear heading hierarchy.
- Omit hero sections, decorative chrome, marketing language, and em dashes.
- Use only the sections needed to communicate the work. Prefer summaries, facts, decisions, risks, requirements, and next steps.

## HTML specification

- Start with `<!doctype html>` and set a meaningful `lang` attribute.
- Include UTF-8 charset metadata and a responsive viewport:

  ```html
  <meta name="viewport" content="width=device-width, initial-scale=1">
  ```

- Use semantic HTML and logical landmarks. Keep the document useful when styles do not load.
- Use inline CSS. Keep the layout fluid and responsive. Do not use fixed-width page or content layouts.
- Use true black (`#000`) for primary text. Use dark gray only for secondary surfaces or accents.
- Use inline SVG for diagrams and icons when needed.
- Use only `https://` or `data:` image sources.
- Keep the file self-contained and independent of frameworks, build steps, routes, components, and application assets.

## Script and link specification

- Use an inline classic script only when interactivity materially improves the writeup.
- Keep scripted pages useful without JavaScript.
- Do not rely on storage, `fetch`, workers, frames, forms, or popups. The sandbox blocks them.
- Do not use external scripts or module scripts.
- In a script-free file, give every external link `target="_blank"` and `rel="noopener noreferrer"`.
- If any script exists, omit `target="_blank"` from links.

## UI Mocks

When the user asks what a UI should look like, use `$prototype`'s UI branch and follow its `UI.md` guidance. Apply the same rules to the standalone HTML artifact:

- Generate three radically different variants by default. Cap the set at five.
- Keep all variants on one HTML route and switch them with the `?variant=` URL parameter.
- Add a floating bottom switcher with previous and next controls, the current variant label, URL updates, and keyboard support for the left and right arrow keys. Do not intercept those keys while an input, textarea, or contenteditable element is focused.
- Make variants structurally different. Change the layout, information hierarchy, and primary affordance. Do not create color-only or copy-only variations.
- Keep the mock read-only and throwaway. Use in-memory state only, avoid real mutations, and keep the mock clearly marked as a prototype.
- Surface the file URL and the available `?variant=` keys so the user can review each option.
- If the mock is later moved into a codebase, prefer mounting it on an existing page and hide the switcher in production builds. Capture the selected variant and remove the losing variants and switcher from production code.

## Workflow

1. Establish the audience, purpose, source material, and required outcome. Ask only when missing information would materially change the document.

2. Draft the content before styling. Preserve the user's facts, dates, metrics, links, and terminology. Mark assumptions and do not invent evidence or decisions.

3. Build the complete HTML file according to the specifications above. Validate structure, heading order, mobile readability, contrast, link behavior, image sources, and the absence of placeholder text.

4. Save one clearly named `.html` file outside the repository's source tree. Keep application code and configuration unchanged.

5. Publish a new draft through Postplan:

   ```sh
   npx postplan upload --new --description "Short description" path/to/writeup.html
   ```

   Use `--draft <draft-id>` to update a specific draft. Use `npx postplan auth login` for local authentication and let the user provide credentials in their own environment. Do not request or store API keys in the conversation or HTML file.

6. After a successful upload, automatically invoke `$postplan-read` with the returned Postplan public URL or raw HTML URL. Use it to confirm that the published document loads and matches the intended content. Report any fetch failure or content mismatch before completing the task.

7. Treat every upload as public unless the service configuration explicitly says otherwise. Confirm before publishing confidential, personal, or restricted content. Return the Postplan public URL and raw HTML URL when the CLI provides them.

Complete the task when the standalone HTML file follows this specification, remains useful without JavaScript, passes the local checks, is verified through `$postplan-read`, and has been published with its resulting URL reported to the user.
