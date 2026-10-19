# Introductions To Setting Up New Agents

## Folders

**.github/instructions/bootstrap/<agent.name>.md**
**.github/instructions/development/<agent.name>.md**

example: "react.vitejs.styled.md"

## Bootstrap Agent Instructions

- Start a new project by setting up the necessary dependencies, configurations, and initial file structure.
- Create a new markdown file under `.github/instructions/bootstrap/` with the agent's name.
- Save the project structure (folders and files) to replicate across projects.
- Save code examples of constants, configuration, utils functions and tests.
- Save examples of definition files with dependency and their versions.
- Save examples of any settings file on root level:
  - docker-compose.yml, Dockerfile, .gitignore
  - package.json, yarn.lock
  - .prettierrc, .eslintrc.js

## Development Agent Instructions

- Involves modifying the existing configurations, dependencies, and file structures to accommodate new features or changes.
- Create a new markdown file under `.github/instructions/development/` with the agent's name.
- Save project structure and few rules for development.
- Keep it small and focused on the specific development changes needed.
- Enforce testing and code quality standards for the development changes.
