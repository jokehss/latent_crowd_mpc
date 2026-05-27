# AGENTS.md

## Role of Codex

You are the implementation agent for this research codebase.

## Rules

- Make small, testable, reviewable changes.
- Do not commit API keys, secrets, datasets, checkpoints, logs, or local absolute paths.
- Do not change evaluation metrics unless explicitly requested.
- Do not change baseline behavior unless explicitly requested.
- Prefer config-driven code.
- Add tests when possible.
- Explain what files changed and why.

## Research Context

This project studies latent world models, MPC, and crowded robot navigation.
The long-term goal is reproducible code for top-tier embodied navigation / robotics research.
