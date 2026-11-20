# Kling AI Agent Skill

The official Kling AI skill for AI agents. Use Kling's CLI and regional MCP services to create images and videos, manage reusable Elements and motion workflows, and check generation tasks.

## Quick start

Install the skill:

```bash
npx skills add klingai-tech/skills
```

Install the CLI for the region of your Kling account, then sign in:

```bash
# Global account
npm i -g @klingai/cli-global --registry=https://registry.npmjs.org

# China account: use this package instead of the Global package
npm i -g @klingai/cli-cn --registry=https://registry.npmjs.org

kling login
```

The Global and China services use separate accounts and credits. Choose the package that matches your Kling account.

## What you can do

- Generate and edit images, and create videos from text, images, references, or motion.
- Create and manage reusable Elements and motion assets.
- Upload local assets and check generation tasks and account credits.

## Official Kling integrations

| Integration | Link |
| --- | --- |
| MCP Plugin — Global | [`https://kling.ai/mcp/plugin`](https://kling.ai/mcp/plugin) |
| MCP Plugin — China | [`https://klingai.com/mcp/plugin`](https://klingai.com/mcp/plugin) |
| Claude Code Plugin — Global | [klingai-tech/claude-plugin](https://github.com/klingai-tech/claude-plugin) |
| Claude Code Plugin — China | [klingai-tech/claude-plugin-cn](https://github.com/klingai-tech/claude-plugin-cn) |
| Cursor Plugin | [klingai-tech/cursor-plugin](https://github.com/klingai-tech/cursor-plugin) |
| ChatGPT Plugin | [klingai-dev/chatgpt-plugin](https://github.com/klingai-dev/chatgpt-plugin) |
| OpenClaw Plugin | [klingai-dev/openclaw-plugin](https://github.com/klingai-dev/openclaw-plugin) |

See the [official MCP setup guide](https://kling.ai/app/mcp/guide) for client-specific setup and capabilities.
