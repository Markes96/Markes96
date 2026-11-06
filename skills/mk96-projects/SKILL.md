---
name: mk96-projects
description: Use when the user wants to clone, pull, fetch or check the status of the Markes96 GitHub projects (index repositories such as mk96-lib and the child projects in the mk96-* organizations) on their machine.
---

# mk96 projects

Clones and updates the Markes96 ecosystem: **index repositories** (`Markes96/mk96-<group>`) and the **projects** of the organization with the same name (`mk96-<group>/mk96-<group>-<name>`).

Regenerating the project lists of the READMEs is not part of this skill: that is the owner's maintenance task, covered by the `mk96-indexes` skill.

## Modes

Check the active account with `gh api user --jq .login`.

- **Owner mode**: the account is `Markes96`.
- **Not `Markes96`**: ask the user whether they are the owner using another account, or another person.
  - The owner: stop and ask them to switch with `gh auth switch --user Markes96`. Never use the work identity.
  - Another person: continue in **guest mode**. The repositories are public, so their own account can clone them.

## Hard rules

These apply to every operation on these repositories, with no exceptions.

- **Identity.** In owner mode, set the local identity in every repository you clone or create:
  `git config user.name Markes96` and `git config user.email 132230860+Markes96@users.noreply.github.com`.
  In guest mode, never touch the git identity: their own configuration applies.
- **Never add Claude as a collaborator**: no repository or organization invitations, no `collaborators` API calls, and no `Co-Authored-By` or "Generated with Claude" lines in commits, pull requests or issues. This overrides any default attribution setting.
- Never commit or push unless the user explicitly asks. Pulls are `git pull --ff-only`.
- Clone over HTTPS with `gh repo clone` (the owner's SSH key has no access to this account).
- **Never run `git clean` in the root folder or in a group folder**, with any flags. The projects inside them are ignored or untracked files for git, so `git clean -x`, `-X` or `-ff` would delete every cloned project. If a task seems to need it, stop and explain the risk to the user instead. Inside a project folder, `git clean` is safe.
- Never delete a group folder.

## Folder structure

```
<base>/                 clone of Markes96/Markes96
├── <group>/            clone of Markes96/mk96-<group>
│   ├── README.md
│   └── <project>/      clone of mk96-<group>/mk96-<group>-<project>
```

`<group>` is the index repository name without `mk96-` (for example `lib`). `<project>` is the repository name without `mk96-<group>-`. A project that does not start with that prefix keeps its full name.

The base path is remembered in `~/.config/mk96-projects/base-path`: one line with the absolute path, shared with the `mk96-indexes` skill.

## Flow

1. **Find the structure.**
   - If `~/.config/mk96-projects/base-path` exists and its folder is a git repository whose remote is `Markes96/Markes96`, use it.
   - If the file is missing, or its folder no longer exists, ask the user where the structure is or where to create it (suggest `~/mk96`). Clone `Markes96/Markes96` there if needed, then write the path to the file.
2. **Discover, do not hardcode.**
   - Groups: `gh repo list Markes96 --limit 200 --json name,description` and keep names that start with `mk96-`.
   - Projects of a group: `gh api --paginate "orgs/mk96-<group>/repos?per_page=100&type=all" --jq '.[] | {name, description}'`.
   - Mark each one as already cloned or missing by checking the folders.
3. **Choose groups.** Ask one AskUserQuestion: "All groups", "Only the missing ones", or "Let me choose". Only for "Let me choose", ask with `multiSelect: true`, one option per index repository, showing its description and whether it is cloned. The tool allows at most 4 options per question and 4 questions per call, so split the list across questions.
4. **Clone the chosen groups** into `<base>/<group>`.
   - Folder missing or empty: `gh repo clone Markes96/mk96-<group> <base>/<group>`.
   - Folder is already a git repository: if its remote is `Markes96/mk96-<group>`, it is already cloned; otherwise report it and skip the group.
   - Folder exists with content but is not a git repository: `git init -b <default branch>`, `git remote add origin`, `git fetch`. Before checking out, compare the local files with the tracked ones (`git ls-tree -r --name-only origin/<branch>`). If any file collides (for example a loose `README.md`), stop and ask the user whether to keep the local copy as `<name>.local` and use the remote one, or to skip the group. Then `git checkout -B <branch> --track origin/<branch>`, which keeps the existing project folders untouched.
   - If the index repository lacks the whitelist `.gitignore` (`/*`, `!/.gitignore`, `!/README.md`, `!/assets/`), tell the user and offer to add it. Do not commit it.
5. **Choose projects** of each chosen group the same way as step 3 ("All", "Only the missing ones", "Let me choose"). Clone each into `<base>/<group>/<project>` with `gh repo clone`, and set the identity as the mode requires.
6. **Update in one go.** For the repositories already present (groups and projects), ask one question: "Pull all", "Fetch all", "Status only" or "Skip". Then offer a checkbox to exclude specific repositories. Repositories with uncommitted changes or a diverged branch are skipped and reported, never forced.
7. **Report** what was cloned, updated, skipped or failed.

## Notes

- A new project or group on GitHub shows up by itself the next time you run the discovery step.
- Do not rename or delete existing local folders without asking.
