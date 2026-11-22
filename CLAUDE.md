# dev-study-archive

Lecture study notes. Each topic is one Archify explainer page, written in Korean.

## Adding a new topic

1. Read the lecture page and its code. Clone the code repo into the scratchpad, not into this repo.
2. Build the page with the `archify` skill (explainer, `@meta locale=ko`). Pass `--repo-root <clone>` when the draft uses `@code`. Work in `.archify/` (gitignored).
3. Copy the result into `<topic-slug>/index.html` and the draft into `<topic-slug>/source.archify.md`.
4. Add a row to the table of contents in `README.md`.
5. Check that no local paths (`/Users/...`, `/private/tmp/...`) ended up in the committed files, then commit and push.

## Editing an existing topic

Edit `<topic-slug>/source.archify.md`, rerun `finalize explainer` on it, and replace `index.html` with the new output.
