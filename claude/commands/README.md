# Commands

This directory contains command definitions for Claude Code. Commands are shortcuts that can be invoked to execute specific workflows or skills.

## Purpose

Commands provide a simple interface to invoke complex workflows or skills. Each command is defined in a markdown file with frontmatter metadata.

## Organization

- Each command is defined in its own `.md` file
- Command files include:
  - `name`: The command name (used for invocation)
  - `description`: Brief description of what the command does
  - Instructions on how to use the command and which skills it invokes

## Available Commands

- `git-commit.md`: Invokes the git-commit skill to create commits with meaningful messages following project standards
