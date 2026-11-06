# Markes96

Personal GitHub account. Every project lives in a themed organization (`mk96-<group>`), and each organization has an **index repository** in this account with the same name (`Markes96/mk96-<group>`) that describes its projects.

## Groups

<!-- groups:start -->
| Group | Index repository | Organization | What it holds |
|---|---|---|---|
| `ai` | [mk96-ai](https://github.com/Markes96/mk96-ai) | [mk96-ai](https://github.com/mk96-ai) | Artificial intelligence projects |
| `api` | [mk96-api](https://github.com/Markes96/mk96-api) | [mk96-api](https://github.com/mk96-api) | APIs and backends |
| `app` | [mk96-app](https://github.com/Markes96/mk96-app) | [mk96-app](https://github.com/mk96-app) | Desktop and system applications |
| `bot` | [mk96-bot](https://github.com/Markes96/mk96-bot) | [mk96-bot](https://github.com/mk96-bot) | Bots |
| `docs` | [mk96-docs](https://github.com/Markes96/mk96-docs) | [mk96-docs](https://github.com/mk96-docs) | Documentation |
| `game` | [mk96-game](https://github.com/Markes96/mk96-game) | [mk96-game](https://github.com/mk96-game) | Games |
| `infra` | [mk96-infra](https://github.com/Markes96/mk96-infra) | [mk96-infra](https://github.com/mk96-infra) | Infrastructure, deployment and configuration |
| `lab` | [mk96-lab](https://github.com/Markes96/mk96-lab) | [mk96-lab](https://github.com/mk96-lab) | Experiments and proofs of concept |
| `lib` | [mk96-lib](https://github.com/Markes96/mk96-lib) | [mk96-lib](https://github.com/mk96-lib) | Reusable libraries |
| `qa` | [mk96-qa](https://github.com/Markes96/mk96-qa) | [mk96-qa](https://github.com/mk96-qa) | Technical tests and Postman collections |
| `web` | [mk96-web](https://github.com/Markes96/mk96-web) | [mk96-web](https://github.com/mk96-web) | Websites |
<!-- groups:end -->

The list of projects of each group is kept in the README of its index repository.

## Local folder structure

Each group is a clone of its index repository, in a folder named after the group. Its projects are cloned inside it, with the group prefix removed from the folder name:

```
mk96/                                  clone of Markes96/Markes96 (this repository)
├── README.md
├── .gitignore
├── .claude-plugin/                    plugin manifest, so the skills install with /plugin
├── skills/
│   ├── mk96-projects/SKILL.md         clone and update
│   └── mk96-indexes/SKILL.md          regenerate the README lists (owner only)
├── lib/                               clone of Markes96/mk96-lib
│   ├── README.md
│   ├── java-common-code/              clone of mk96-lib/mk96-lib-java-common-code
│   └── java-spring-extensible/        clone of mk96-lib/mk96-lib-java-spring-extensible
└── app/                               clone of Markes96/mk96-app
    ├── README.md
    └── debian-rocket-tool/            clone of mk96-app/mk96-app-debian-rocket-tool
```

Each index repository only tracks its own `README.md`. A whitelist `.gitignore` (ignore everything, allow only the README) keeps the cloned projects out of it, so adding a project never requires editing the ignore file:

```gitignore
/*
!/.gitignore
!/README.md
!/assets/
```

A project whose name does not start with its group prefix keeps its full name.

> **Warning:** never run `git clean` in the root folder or in a group folder. For git the cloned projects are ignored or untracked files, so `git clean -x`, `-X` or `-ff` deletes all of them. Inside a project folder it is safe.

## Cloning everything

This repository is also a Claude Code plugin with the `mk96-projects` skill, which clones and updates the whole ecosystem. Install it with:

```
claude plugin marketplace add Markes96/Markes96
claude plugin install mk96-projects@markes96
```

Then ask Claude Code to "clone my mk96 projects". It will:

1. Look for the folder structure, and ask where you want it the first time. The answer is remembered in `~/.config/mk96-projects/base-path`.
2. Discover the groups and projects on GitHub, and ask whether you want all of them, only the missing ones, or to pick them with checkboxes.
3. Clone the selected groups, then the selected projects of each group.
4. Update what is already cloned in one go (pull, fetch or status), letting you exclude specific repositories.

With the `Markes96` account, every clone gets the owner's git identity. Anyone else can use the plugin too: the repositories are public, so it clones them with their own account and leaves their git identity untouched. Claude is never added as a collaborator.

## Maintaining the lists

The `mk96-indexes` skill, for the owner only, regenerates the project lists of the index READMEs and the group table above from GitHub. Ask Claude Code to "refresh the mk96 indexes". It shows the changes, and commits only if you ask.
