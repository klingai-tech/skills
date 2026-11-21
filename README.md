# Kling AI Skills and developer integrations

Install the official Kling CLI Skill in a compatible coding agent:

```bash
npx skills add klingai-tech/skills
```

The Skill uses the Kling CLI. Choose the package for the region of your Kling account; the Global and China accounts, credits, and tasks are separate.

| Region | Official plugin source | Copilot CLI | VS Code | Standalone CLI | MCP endpoint |
| --- | --- | --- | --- | --- | --- |
| Global (`kling.ai`) | [Kling AI Global Plugin](https://github.com/klingai-tech/claude-plugin) · [npm package](https://www.npmjs.com/package/@klingai/plugin-global) | `copilot plugin install klingai-tech/claude-plugin` | [Install in VS Code](https://vscode.dev/redirect?url=vscode%3A%2F%2Fchat-plugin%2Finstall%3Fsource%3Dklingai-tech%252Fclaude-plugin%26plugin%3Dkling-ai-global-plugin) | `npm i -g @klingai/cli-global --registry=https://registry.npmjs.org` | `https://kling.ai/mcp/plugin` |
| China (`klingai.com`) | [Kling AI China Plugin](https://github.com/klingai-tech/claude-plugin-cn) · [npm package](https://www.npmjs.com/package/@klingai/plugin-cn) | `copilot plugin install klingai-tech/claude-plugin-cn` | [Install in VS Code](https://vscode.dev/redirect?url=vscode%3A%2F%2Fchat-plugin%2Finstall%3Fsource%3Dklingai-tech%252Fclaude-plugin-cn%26plugin%3Dkling-ai-cn-plugin) | `npm i -g @klingai/cli-cn --registry=https://registry.npmjs.org` | `https://klingai.com/mcp/plugin` |

## Install the plugin in an agent

- **Claude Code:** follow the install commands in the matching regional plugin repository above.
- **[GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/plugins-finding-installing):** install directly from the matching GitHub repository with the command in the table.
- **[VS Code with Copilot](https://code.visualstudio.com/docs/agent-customization/agent-plugins):** use the matching install link in the table. VS Code asks you to confirm before registering the source and installing the plugin. If the link does not open, run **Chat: Install Plugin from Source** and enter the matching repository URL above.
- **Other MCP clients:** use the matching `/mcp/plugin` endpoint from the table and complete OAuth with the same region's Kling account.

VS Code plugin links require Copilot Agent Host support and the `chat.plugins.enabled` setting. The first connection uses the selected region's OAuth flow.

Generation uses credits from the connected regional Kling account. Check the official regional guide for client-specific setup, supported models, and usage details:

- [Global Kling MCP and CLI guide](https://kling.ai/app/mcp/guide)
- [China Kling MCP and CLI guide](https://klingai.com/app/mcp/guide)
