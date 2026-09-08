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

From your existing repository:

```sh
aops init --project-slug your-project-slug
```

Replace `your-project-slug` with your project name, such as `my-app`. This initializes the repository's local aops settings. If the project does not exist on your aops server yet, ask your agent to create it and link this repository.

Then ask your agent:

| What you need | A simple prompt |
| --- | --- |
| Save your progress | “aops mem slug:your-project-slug, create a checkpoint for this work.” |
| Continue later | “aops mem slug:your-project-slug, resume where we left off.” |
| Plan the work | “aops pm slug:your-project-slug, use a sprint for this task.” |
| Create a document | “aops doc slug:your-project-slug, create a document for this project.” |
| Find your documents | “aops doc slug:your-project-slug, list the documents in the architecture group.” |

Start with one capability. Add a working discipline when you want a repeatable solo or collaborative workflow. Explore the [public assets](assets/) or visit [aopslab.com](https://aopslab.com) for more.
