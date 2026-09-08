# aops

Start at **[aopslab.com](https://aopslab.com)** for an introduction and getting-started guides.

aops gives your AI agents shared project memory, plans, and versioned documents—so they can pick up where you left off.

This repository contains the public aops **user guides, skills, working disciplines, and roles** in [`assets/`](assets/). These help agents use aops; you do not need to clone this repository to get started.

## Install and set up

With Node.js and npm installed, run:

```sh
npm install --global @aopslabs/aops
aops
```

`aops` opens the terminal setup interface (TUI). `aops-cli` is an alternative command name. Choose SQLite for a simple local setup, or enter your existing PostgreSQL connection. Configure the aops Server connection and check that setup is ready. You do not need to install `@aopslabs/aops-server` separately.

## Use it in your project

Open your existing repository in your agent and try one of these prompts. Replace `your-project-slug` with your project name, such as `my-app`.

| What you need | A simple prompt |
| --- | --- |
| Initialize your project | “aops init slug:your-project-slug, initialize this repository and create or link its project on the aops server.” |
| Save your progress | “aops mem slug:your-project-slug, create a checkpoint for this work.” |
| Continue later | “aops mem slug:your-project-slug, resume where we left off.” |
| Plan the work | “aops pm slug:your-project-slug, use a sprint for this task.” |
| Create a document | “aops doc slug:your-project-slug, create a document for this project.” |
| Find your documents | “aops doc slug:your-project-slug, list the documents in the architecture group.” |
| Record a problem | “aops pm slug:your-project-slug, create an issue for the problem we just encountered.” |
| Discuss a decision | “aops discuss slug:your-project-slug, open a Claude session in tmux and lead an independent discussion with it about SQLite or PostgreSQL for this project, following the aops-cli-discuss skill.” |
| Work with a reviewer | “Use aops-collaborative-work for slug:your-project-slug, with Codex as implementer and Fable as reviewer.” |
| Leave a review for later | “aops pm slug:your-project-slug, create a review request for another agent to review this work later.” |

Start with one capability. Add a working discipline when you want a repeatable solo or collaborative workflow. Explore the [public assets](assets/) or visit [aopslab.com](https://aopslab.com) for more.
