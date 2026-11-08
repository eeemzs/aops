# aops

The all-in-one agentic development tool. Memory, plans, documents, reviews and chat in one place, so your agents pick up where they left off.

**[Download aops at aopslabs.com](https://aopslabs.com)** for Windows, Mac (Apple silicon) or Linux.

- Free forever, even for paid work. No account and no API key.
- Runs on your computer, and your projects stay there.
- Sets itself up for Codex and Claude. Any other agent or harness that can run a command can use aops too.
- Team and server editions are coming soon.

## What is in this repository

The program is free to use but not open source. This repository holds the public aops **skills, user guides, working disciplines and roles** in [`assets/`](assets/). The installer brings them with it, so you do not need to clone this repository. You may change the skills and create your own.

## Get newer skills

aops installs the skills that come with it. Newer skills are published here as `assets-<version>` releases. The Skills row of the aops window checks for them, or run:

```sh
aops assets status --check-latest
aops assets update
```

An update leaves a skill you changed as it is and tells you which one; the other skills are updated.

## Use it in your project

Open your repository in your agent and try these prompts. Replace `slug:my-app` with your project's short name.

| What you need | Prompt |
| --- | --- |
| Create the project | “Use aops init to set up this repository as slug:my-app and create or link its project on the aops server.” |
| Learn how to work with aops | “How do I use aops with you on this project?” |
| Get a brief | “Use aops mem to brief me on where slug:my-app stands.” |
| Write a checkpoint | “Use aops mem to write a checkpoint for slug:my-app with what was checked and the next step.” |
| Resume work | “Use aops mem to resume this task in slug:my-app from the latest checkpoint.” |
| Plan a task | “Use aops pm to turn this task into a sprint plan for slug:my-app, with phases and small steps.” |
| Work with a reviewer | “Use the aops collaborative work discipline. You are the implementer and driver; the other agent is the reviewer. Task: <describe the task here>.” |
| Write your own working discipline | “Write my working discipline as a skill named my-way, install it for Codex and Claude, and name it in AGENTS.md or CLAUDE.md: <rules>.” |
| Leave a review for later | “Use aops pm to open a review request in slug:my-app so another agent can review this work later.” |
| Record a decision | “Use aops doc to record this decision in the slug:my-app decision log.” |
| Discuss a decision | “aops discuss slug:my-app, open a Claude session in tmux and lead an independent discussion with it about SQLite or PostgreSQL for this project, following the aops-cli-discuss skill.” |

More prompts, using aops from another computer over SSH, and uninstalling are in the [guide](https://aopslabs.com/docs).

## Questions and bugs

Open an issue in [this repository](https://github.com/eeemzs/aops/issues). News is on [X](https://x.com/eeemzs).

## License

The free edition, including the skills and guides in this repository, comes under the [aops Free Edition License](https://aopslabs.com/license): free for any purpose, including commercial work. You may change the skills and create your own.
