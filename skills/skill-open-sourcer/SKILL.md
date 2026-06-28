---
name: skill-open-sourcer
description: Package and publish a local agent skill as an open-source GitHub repository. Use when the user gives a local SKILL.md path or skill directory and asks to open-source, publish, release, share, make installable with npx skills, prepare README/docs, create a public skill repo, or generate launch/social copy for a local Codex, Agents, Claude, or compatible skill.
---

# Skill Open Sourcer

Turn one local skill into a clean public release package. The normal input is only a `SKILL.md` path or a skill directory.

Default posture: move fast, but stop on release risk. If the safety scan finds secrets, private paths, client data, unpublished proprietary material, or ambiguous ownership, report the blockers and do not publish until the user resolves them.

## Quick Start

1. Resolve the input to a real skill directory. Accept a `SKILL.md` file, a folder containing `SKILL.md`, or a symlink under `.codex/skills`, `.agents/skills`, `.claude/skills`, or another local skill root.
2. Run the environment check before packaging or publishing:

```bash
SKILL_OPEN_SOURCER_DIR="${CODEX_HOME:-$HOME/.codex}/skills/skill-open-sourcer"
python3 "$SKILL_OPEN_SOURCER_DIR/scripts/check_release_env.py"
```

Required local commands are `python3`, `git`, `node`, and `npx`. `gh` is the default local GitHub publishing surface, but GitHub MCP/app or an existing authenticated remote can replace it. If no GitHub publishing surface is available, build and validate the repo locally, then stop with the missing setup.

3. Read the skill entrypoint and inspect directly referenced `scripts/`, `references/`, `assets/`, and `agents/` files. Do not copy unrelated local folders.
4. Run the safety scanner:

```bash
SKILL_OPEN_SOURCER_DIR="${CODEX_HOME:-$HOME/.codex}/skills/skill-open-sourcer"
python3 "$SKILL_OPEN_SOURCER_DIR/scripts/scan_skill_release.py" /path/to/skill-or-SKILL.md
```

5. If the environment and safety scans are clear, create a fresh release repository outside the source skill directory.
6. Copy the skill into `skills/<skill-name>/`, preserving only files needed for the skill to work.
7. Add the public release files described in `references/release-package.md`, including the README quality gate and GitHub metadata recommendations.
8. Validate the packaged skill and `npx skills` listing before publishing.
9. Publish to GitHub when low-risk checks pass, then produce launch copy.

Read `references/release-package.md` before writing the release repo.

## Output Contract

For a successful release, return:

- GitHub repo URL.
- Install command using `npx skills`.
- Verification results for skill validation and `npx skills add <owner>/<repo> --list`.
- Short risk summary: what was scanned, what was removed or sanitized, and any residual assumptions.
- GitHub Description and Topics recommendations when creating a new public repo.
- Launch copy: at least one X/Twitter post, plus optional Chinese version when the user works in Chinese.

If blocked, return:

- Blocker list with file paths and reasons.
- Suggested sanitization steps.
- The safest next action, usually "clean these files, then rerun the release".

## Safety Rules

Hard stop when any of these appear in the source package:

- API keys, tokens, private keys, cookies, credentials, `.env` files, or auth config.
- Absolute personal paths such as `/Users/...`, `/home/...`, local vault paths, or machine-only cache paths that would confuse public users.
- Customer names, private company data, internal docs, private URLs, unpublished prompts, or paid/proprietary assets without a clear license.
- Symlinks that escape the package, large binaries, databases, logs, browser profiles, `.DS_Store`, `__pycache__`, or generated cache files.
- Unclear license or ownership for bundled code/assets.

Low-risk auto publish is allowed only when the scanner and manual review find no blockers. If the target GitHub repo already exists with unrelated content, stop and ask before overwriting or force pushing.

## Packaging Rules

- Keep the original skill behavior intact. Refactor only enough to remove private assumptions and make public installation reliable.
- Prefer a lean `SKILL.md`; move detailed public guidance into `references/` only when an agent will genuinely need it.
- Do not add a README inside the skill folder. Put human-facing documentation at the release repo root.
- Preserve `agents/openai.yaml` when present. If absent, create it with `display_name`, `short_description`, and a `default_prompt` that explicitly mentions `$<skill-name>`.
- Include `scripts/` only for deterministic helper logic the skill actually uses.
- Include `assets/` only when licensing and size are safe for public release.
- Use MIT as the default license only when the source skill has no conflicting license and ownership is clear.

## Publication Flow

1. Run `scripts/check_release_env.py`. Add `--repo-dir <release-repo>` when updating an existing local repo. Add `--check-npx-skills` when package/network access is uncertain.
2. If required commands are missing, stop and report the exact blockers. If only `gh` is missing, continue only when GitHub MCP/app is available or the release repo already has an authenticated `origin` remote.
3. Build the release repo locally.
4. Validate with the system skill validator when available:

```bash
python3 "${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-creator/scripts/quick_validate.py" skills/<skill-name>
```

5. Create or update the GitHub repository using the available GitHub surface (`gh`, GitHub MCP/app, or existing remote). Avoid force push.
6. Push the branch or `main`.
7. Verify listing:

```bash
npx skills add <owner>/<repo> --list
```

8. If listing works, provide the install command:

```bash
npx skills add <owner>/<repo> -g -a codex --skill <skill-name> -y
```

Adjust the agent flag and install wording when the target platform is not Codex.

## Launch Copy

After a successful release, draft concise social copy that explains:

- The problem the skill solves.
- What a user gives it.
- What it publishes or automates.
- The install command or repo link.
- One concrete example request.

For Chinese users, include a natural Chinese launch post. Keep it human-facing; do not over-explain scripts.
