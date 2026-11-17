# Employee Leave Management System

An educational college final project for designing and researching an employee leave management system.

This project is **not intended to be production software for companies**. Its purpose is to help the group learn about system design, collaboration, and security features that could be useful in a leave-management application.

## Project overview

The system is still being researched and designed. Possible areas include:

- Employee leave requests
- Manager approval and rejection
- Leave balances and history
- Administrative management
- Notifications and reporting
- Authentication, authorization, validation, and audit history

The final technologies and features will be decided through research and documented in the repository.

## Getting started

### Get the repository

1. Open the repository on GitHub.
2. Select **Code** and copy the repository URL.
3. Clone it on your computer:

   ```bash
   git clone https://github.com/Wynteriall/leave-system
   cd leave-system
   ```

4. Open the folder in your editor or development environment.

### Update your local copy

Before starting work, update the local `main` branch:

```bash
git switch main
git pull origin main
```

Follow the setup instructions for the technologies selected by the project. These instructions will be added or updated as the research progresses.

## How to contribute

Work should normally begin with a GitHub Issue. Do not start a large change without first checking whether an issue already exists.

1. Open the repository's **Issues** tab.
2. Look for an open issue that interests you and check whether it is already assigned or blocked.
3. If no suitable issue exists, create one describing the question, problem, research task, or feature.
4. Ask questions and discuss the issue before making uncertain decisions.
5. Create a branch from `main` for the work:

   ```bash
   git switch main
   git pull origin main
   git switch -c <short-description>
   ```

6. Make focused changes and update relevant documentation.
7. Commit and push the branch:

   ```bash
   git add .
   git commit -m "Describe the change"
   git push -u origin <short-description>
   ```

8. Open a pull request on GitHub, link it to the issue, and request review.
9. Address review comments and merge only after the group agrees the change is ready.

Keep issues, branches, and pull requests focused on one understandable piece of work.

## Research and project knowledge

Research should be written in Markdown and committed to the repository when it becomes useful to the project. Link research to the GitHub Issue that requested it so that others can find the reasoning later.

Before starting new research:

- Search existing GitHub Issues and repository documentation.
- Check whether the question has already been researched.
- Record sources, findings, assumptions, and unresolved questions.
- Explain how the research affects the system or a future decision.

Important project terminology belongs in [`GLOSSARY.md`](GLOSSARY.md) when that file is created. Significant technical decisions belong in [`docs/adr/`](docs/adr/) when applicable. See [`docs/agents/domain.md`](docs/agents/domain.md) for the documentation structure.

## Working with AI agents

Group members may use an AI agent such as ChatGPT, Claude, Gemini, or another coding assistant. The person using the agent remains responsible for reviewing its work.

Use a prompt like this:

> You are contributing to the Employee Leave Management System, an educational college project. First read `README.md`, `AGENTS.md`, the relevant GitHub Issue, and related documentation. Explain your plan before making changes. Keep the work limited to the issue, ask questions when requirements are unclear, follow the repository's existing conventions, and update documentation when needed. Do not invent project requirements. Before finishing, check the changes, run relevant tests or checks, and summarize the files changed, decisions made, and verification performed.

An agent should not close an issue, merge a pull request, or make security-sensitive decisions without the contributor reviewing the result.

## Current repository guidance

- GitHub Issues are the source of truth for project tasks and research questions.
- Read [`docs/agents/issue-tracker.md`](docs/agents/issue-tracker.md) for the repository's issue workflow.
- Read [`docs/agents/domain.md`](docs/agents/domain.md) before changing domain terminology or recording technical decisions.
- Treat security as a learning goal: document risks, assumptions, and limitations rather than claiming the system is production-ready.
