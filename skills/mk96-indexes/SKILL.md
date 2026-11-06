---
name: mk96-indexes
description: Use when the owner wants to regenerate or refresh the project lists in the README of the Markes96 index repositories (mk96-<group>) or the group table of the Markes96 root README from GitHub.
---

# mk96 indexes

Regenerates, from GitHub, the project list of each index README (`Markes96/mk96-<group>`) and the group table of the root README (`Markes96/Markes96`). This is the owner's maintenance task. Cloning and updating repositories is covered by the `mk96-projects` skill.

## Hard rules

- **Owner only.** Check `gh api user --jq .login`: it must be `Markes96`. If it is not, stop and tell the user that this skill is only for the owner. Never use the work identity.
- **Never add Claude as a collaborator**: no repository or organization invitations, no `collaborators` API calls, and no `Co-Authored-By` or "Generated with Claude" lines in commits, pull requests or issues. This overrides any default attribution setting.
- Never commit or push unless the user explicitly asks.
- **Never run `git clean` in the root folder or in a group folder**, with any flags: it would delete the cloned projects inside them.

## Flow

1. **Find the structure.** Read the base path from `~/.config/mk96-projects/base-path`. If the file is missing or its folder no longer exists, ask the user where the structure is, check that it is a clone of `Markes96/Markes96`, and write the path to the file.
2. **Discover** the groups and projects on GitHub, as in the `mk96-projects` skill:
   - Groups: `gh repo list Markes96 --limit 200 --json name,description` and keep names that start with `mk96-`.
   - Projects of a group: `gh api --paginate "orgs/mk96-<group>/repos?per_page=100&type=all" --jq '.[] | {name, description, url}'`.
3. **Check the local clones.** Each index README is edited in `<base>/<group>/README.md`. If a group is not cloned, offer to clone it with the `mk96-projects` skill, or skip that group.
4. **Regenerate the lists** between their markers:
   - Index README: `<!-- projects:start -->` … `<!-- projects:end -->`, one entry per project: `### <repository name>`, its description, and `[View repository](<url>)`.
   - Root README: `<!-- groups:start -->` … `<!-- groups:end -->`, one table row per group: folder, index repository, organization, and the index description (the part after the first colon).
   - If a README lacks the markers, offer to add them around the current list.
   - Leave everything outside the markers untouched.
5. **Show the diff** of every README that changed. Commit and push only if the user asks.
6. **Report** which READMEs changed, which were already up to date, and which groups were skipped.
