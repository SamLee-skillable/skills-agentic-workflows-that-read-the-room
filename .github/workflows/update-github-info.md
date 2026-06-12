---
# Run once a day (fuzzy schedule, scattered execution time) and on demand.
on:
  schedule: daily
  workflow_dispatch:

# Run this workflow on the Claude engine (this repo does not use GitHub Copilot).
engine: claude

# Keep the agent job read-only. All writes happen via safe-outputs.
permissions:
  contents: read

# Tools available to the agent:
# - edit: modify files in the workspace (the proposed change is captured by the
#   create-pull-request safe output, not pushed directly).
# - web-fetch: fetch web pages (github.blog latest + changelog).
tools:
  edit:
  web-fetch:

# Allow outbound network access to github.blog in addition to engine defaults
# so the web-fetch calls below succeed. The "github" ecosystem identifier
# expands to GitHub-owned domains, including github.blog.
network:
  allowed:
    - defaults
    - github

# All writes are routed through safe-outputs. The agent never writes to the repo
# directly; instead it proposes its file changes as a pull request for review.
safe-outputs:
  create-pull-request:
    title-prefix: "[github-info] "
    draft: false
  # If there is nothing new worth reporting, the agent calls noop and exits
  # without opening a pull request.
  noop:
---

# Update GitHub Info for Mona's readers

You maintain `site/content/github-info.md`, a curated summary of recent GitHub
news for Mona's website readers. Your job is to refresh it with new, noteworthy
GitHub updates while staying true to Mona's editorial voice.

## Steps

1. **Read the editorial guidance.** Read `notes/mona-notes.md` and follow it.
   In short: keep summaries short and practical, prefer updates that help
   developers learn GitHub faster, always cite the source (GitHub Blog or
   Changelog), and publish changes through a pull request so Mona can review.

2. **Read the current content.** Read `site/content/github-info.md` so you know
   its existing structure and which updates are already covered. The file has
   these sections, which you MUST preserve:
   - `## Mona's editorial angle`
   - `## Current homepage themes`
   - `## Latest GitHub Updates` (a bulleted list of dated items, each ending
     with a markdown link to its source)

3. **Fetch fresh GitHub news** using `web-fetch`:
   - <https://github.blog/latest/>
   - <https://github.blog/changelog/>

4. **Identify what is genuinely new.** Compare the fetched stories against the
   items already listed under `## Latest GitHub Updates`. Only consider items
   that are not already represented and that fit Mona's angle (practical,
   developer-focused GitHub guidance).

5. **Update `site/content/github-info.md`** (only if there is something new):
   - Add new entries to the top of the `## Latest GitHub Updates` list.
   - Match the existing bullet format exactly:
     `- **Title (Date):** One or two concise, practical sentences. ([changelog](URL))`
     (use `[blog]` when the source is a blog post rather than the changelog).
   - Keep the list focused — trim the oldest entries if it grows beyond about
     6 items so it stays scannable.
   - Do not alter the `## Mona's editorial angle` or
     `## Current homepage themes` sections unless a new theme is clearly
     warranted.
   - Use the `edit` tool to make these changes.

6. **Open a pull request** with the `create-pull-request` safe output so Mona
   can review before anything goes live. Give it a clear title summarizing the
   updates added and a body that lists each new item with its source link.

7. **Nothing new?** If, after comparing, there are no new updates worth
   reporting, make no file changes and call `noop` instead of opening a pull
   request. Do not open an empty or no-op PR.

## Constraints

- Stay strictly within this repository; only edit `site/content/github-info.md`.
- Every update you add must cite its source link (Blog or Changelog).
- Keep the tone concise and practical, consistent with the existing entries.

## Usage

- Runs automatically once per day, and can be triggered manually from the
  Actions tab or with `gh aw run update-github-info`.
- The workflow never commits to `main` directly: it opens a pull request titled
  with the `[github-info]` prefix for Mona to review and merge.
- Requires an `ANTHROPIC_API_KEY` repository secret (the workflow runs on the
  Claude engine).
