---
name: tanzu-onboard
description: Complete workflow for onboarding, modernizing, and upgrading Spring applications on Tanzu Platform. Use when the user asks to onboard, modernize, upgrade, get advice, or apply recommendations for their Spring application on Tanzu Platform or Cloud Foundry.
license: LICENSE.txt
version: 1.6.3
---

## MANDATORY FIRST ACTION — execute before anything else

**STOP. You MUST invoke the `tanzu-shared` skill right now using the Skill tool, before reading any further instructions.** Do not check prerequisites, do not detect the project layout, do not run any commands. Invoke `tanzu-shared` first, then continue with the steps below.

# Tanzu Spring Modernization Workflow

Help the user modernize, upgrade, and onboard their Spring application to Tanzu Platform.

Follow this workflow step by step. Stop and wait for confirmation before applying any changes.

---

## Step 1: Check Prerequisites

Run these commands to verify the environment is set up correctly:

```bash
cf help -a | grep repo
```

If the CF CLI plugin is not installed (no "repo" commands in output), stop and tell the user to install it.

Then run `cf repo -h` to discover the available subcommands. Only use commands and flags shown in this output throughout the workflow.

## Step 2: Detect project layout

Determine whether the project root contains a single application or multiple applications (monorepo).

1. Check whether any of these files exist **at the project root**: `pom.xml`, `build.gradle`,
   `build.gradle.kts`, `package.json`, or COBOL source files (`*.cbl`, `*.cob`).
2. **If a project indicator exists at the root**, this is a single-application project. Set
   `<selected-folder>` to empty and proceed to Step 3.
3. **If no project indicator exists at the root**, scan up to 2 levels deep for subdirectories
   containing any of these indicators: `pom.xml`, `build.gradle`, `build.gradle.kts`,
   `package.json`, `Makefile`, `Dockerfile`, or source files (`*.cbl`, `*.cob`, `*.cobol`).
4. If multiple candidate directories are found, **use the AskQuestion tool** to present them and
   ask the user which application to work with. Set `<selected-folder>` to the user's choice
   (relative path from the project root, e.g. `applications/datasort`).
5. If exactly one candidate directory is found, use it as `<selected-folder>` without asking.
6. If no candidate directories are found, warn the user and proceed without a folder.

Remember `<selected-folder>` for all subsequent `cf repo` commands in this session.

## Step 3: Onboard application

If there is no `manifest.yml` in the project (or in `<selected-folder>` when set), suggest
onboarding the application as the first action.

1. Ask the user: "Would you like to run `cf repo onboard-app`?
2. If the user replies yes, run exactly this command:
   ```bash
   cf repo onboard-app --path=<project-path> --folder=<selected-folder> --non-interactive
   ```
   Omit `--folder` if `<selected-folder>` is empty (single-application project).
   Then show the user the full command output. If any printed challenge line contains `[ai-assisted]`, follow **AI-assisted challenges** (section below).
3. If the user replies no, skip onboard-app and proceed to Step 4.

## Step 4: Get Advice recommendations

1. Run:
   ```bash
   cf repo advice -p <project-path>
   ```
2. Show the results. If any printed challenge or blocker line contains `[ai-assisted]`, or the output includes the **Legend** with an **ai-assisted** bullet, follow **AI-assisted challenges** (section below).

Tell the user what the assessment found:

- What migrations or improvements are recommended?
- Does it need Spring upgrades?

## AI-assisted challenges

After `cf repo onboard-app` (Step 3) or `cf repo advice` (Step 4), inspect the printed **challenge or blocker** lines.

**When to recommend `/tanzu-modernize-app`:** If any line shows the fix type **`[ai-assisted]`** (rows look like `name [ai-assisted]: …`), or the output includes the **Legend** and its **ai-assisted** line (often printed with `cf repo advice` when there are matching challenges), you MUST **recommend that the user run the `/tanzu-modernize-app` skill** as the next step for those items.

**Do not use `cf repo apply-advice` for ai-assisted issues.** The printed Legend describes **auto-fix** as addressable with `cf repo apply-advice`. **ai-assisted** items should be handled via **`/tanzu-modernize-app`**, not `apply-advice`.

**Onboarding vs advice output:** `cf repo onboard-app` may list challenges with `[fixType]` but **not** print the full Legend; in that case rely on **`[ai-assisted]`** in the challenge rows. `cf repo advice` may print both rows and the Legend.

## Step 5: Apply Advices

Once the full list of advices is received in Step 4, if `spring-governance-starter` appears,
ask the user if they want to apply it.

Offer `cf repo apply-advice` only for **auto-fix** (or otherwise CLI-applicable) items from the assessment. For **ai-assisted** items, follow **AI-assisted challenges** above instead of this step.

For each recommendation:

1. Tell the user what the migration does (e.g., "migrate-slf4j: Migrates logging to SLF4J")
2. Ask: "Do you want to apply this migration?"
3. If the user says yes, run:
   ```bash
   cf repo apply-advice -p <project-path> -n <exact-advice-name>
   ```
   Use the exact advice name from the `cf repo advice` output.
4. Show what changed.

## Step 6: Upgrade plan (optional)

If the user wants to upgrade dependencies or build tools:

1. Run:
   ```bash
   cf repo upgrade-plan -p <project-path>
   ```
2. Show the plan (sequence of steps).
3. Ask: "Do you want to apply the first step of the upgrade plan?"
4. If yes, run:
   ```bash
   cf repo apply-upgrade-plan -p <project-path>
   ```
5. Show what changed.

## Summary

After completing all steps, give the user a summary of:

- What was analyzed
- What changes were applied
- What recommendations remain (manual fixes, etc.)
- Whether onboard-app was completed (if applicable)
- Next steps

---

## Command Usage Guidelines

### When to Ask for Confirmation

**Always ask before running:**

- `cf repo apply-advice` - applying migrations
- `cf repo onboard-app` - executing (it applies changes and generates manifest)
- `cf repo apply-upgrade-plan` - applying upgrade plan steps

**No need to ask before:**

- `cf repo advice` - just run and show results
- `cf repo upgrade-plan` - just run and show the plan

**Order:** Offer `cf repo onboard-app` first (Step 3) if no `manifest.yml` exists, before proceeding to advices.

### Showing command output (mandatory)

After running any `cf repo` command, you MUST include the full command output (or a clear, faithful summary)
in your response. Do not only say "command succeeded" or "command failed" -- show the relevant output so the
user can see progress, errors, and results (e.g. assessment steps, manifest path, onboarding finished).

### Parameter Values

For every command, provide the path parameter (`-p` or `--path`).

The `--folder` flag is only supported by `cf repo onboard-app`. If Step 2 identified a
`<selected-folder>` (monorepo), pass `--folder=<selected-folder>` to `cf repo onboard-app` only.
Do NOT pass `--folder` to `cf repo advice`, `cf repo apply-advice`, `cf repo upgrade-plan`, or
`cf repo apply-upgrade-plan` -- they do not support it.
To execute these commands, change directories before running the command temporarily.

DO NOT pass other optional parameters (build tool, JVM args, etc.) unless the user explicitly provides them.

For `cf repo apply-advice`, use the exact advice name from `cf repo advice` output with `-n`.

For `cf repo onboard-app`, always use `--non-interactive`.

### When a command fails

If any `cf repo` command fails:

1. Show the full error output to the user
2. Tell the user the command failed and suggest they retry later or check their environment
3. Do NOT suggest running other commands to work around the failure
4. Do NOT invent commands (e.g. `cf repo assess`) or flags that are not in the `cf repo -h` output
5. Do NOT suggest installing a different plugin version
6. Just report the failure and stop

---

Start now by checking the prerequisites (Step 1).
