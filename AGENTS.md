cat > AGENTS.md <<'EOF'
# AGENTS.md

## Role of Codex

You are the implementation agent for a research codebase on crowded embodied navigation and world models.

## Rules

- Make small, reviewable changes.
- Do not change baselines unless explicitly requested.
- Do not change evaluation metrics unless explicitly requested.
- Do not commit API keys, secrets, local paths, or datasets.
- Add tests for new modules whenever possible.
- Prefer clear, reproducible, config-driven code.
- Before finishing, explain what files changed and why.

## Research Standard

Every code change should support reproducible experiments for a potential top-tier ML/robotics paper.
EOF