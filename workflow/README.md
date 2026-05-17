# Workflow

Place your exported n8n workflow JSON in this directory, named `research-agent.json`.

## How to export from n8n

1. Open your working workflow in n8n
2. Top-right `…` menu → **Download** (or **Export**)
3. Save the resulting `.json` file as `research-agent.json` in this directory
4. Commit and push

## What this gives users who clone the repo

Anyone with an n8n account can import this JSON via **Workflows → Import from File** and get a fully configured workflow shape (nodes, connections, expressions). They'll still need to set their own credentials (API keys) before it runs.

## Note

The exported workflow does not include credentials — n8n strips those during export, which is what you want for a public repo.
