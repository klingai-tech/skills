# Kling AI Skills and developer integrations

[![skills.sh installs](https://skills.sh/b/klingai-tech/skills)](https://skills.sh/klingai-tech/skills)

Install the official Kling CLI Skill in a compatible coding agent:

```bash
npx skills add klingai-tech/skills
```

The Skill uses the Kling CLI. Choose the package for the region of your Kling account; the Global and China accounts, credits, and tasks are separate.

| Region | Official plugin source | Copilot CLI | VS Code | Standalone CLI | MCP endpoint |
| --- | --- | --- | --- | --- | --- |
| Global (`kling.ai`) | [Kling AI Global Plugin](https://github.com/klingai-tech/claude-plugin) · [npm package](https://www.npmjs.com/package/@klingai/plugin-global) | `copilot plugin install kling-ai@klingai` | [Install in VS Code](https://vscode.dev/redirect?url=vscode%3A%2F%2Fchat-plugin%2Finstall%3Fsource%3Dklingai-tech%252Fclaude-plugin%26plugin%3Dkling-ai) | `npm i -g @klingai/cli-global@latest --registry=https://registry.npmjs.org` | `https://kling.ai/mcp/plugin` |
| China (`klingai.com`) | [Kling AI China Plugin](https://github.com/klingai-tech/claude-plugin-cn) · [npm package](https://www.npmjs.com/package/@klingai/plugin-cn) | `copilot plugin install kling-ai-cn@klingai-cn` | [Install in VS Code](https://vscode.dev/redirect?url=vscode%3A%2F%2Fchat-plugin%2Finstall%3Fsource%3Dklingai-tech%252Fclaude-plugin-cn%26plugin%3Dkling-ai-cn) | `npm i -g @klingai/cli-cn@latest --registry=https://registry.npmjs.org` | `https://klingai.com/mcp/plugin` |

## Install the npm plugin package

These packages install the full plugin and its local CLI. npm requires explicit approval for their install scripts:

- Global: `npm i -g @klingai/plugin-global@latest --registry=https://registry.npmjs.org --allow-scripts=@klingai/plugin-global`
- China: `npm i -g @klingai/plugin-cn@latest --registry=https://registry.npmjs.org --allow-scripts=@klingai/plugin-cn`

## Install the plugin in an agent

- **Claude Code:** follow the install commands in the matching regional plugin repository above.
- **[GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/plugins-finding-installing):** add the matching official marketplace, then install the plugin named in the table: `copilot plugin marketplace add klingai-tech/claude-plugin` followed by `copilot plugin install kling-ai@klingai` for Global, or `copilot plugin marketplace add klingai-tech/claude-plugin-cn` followed by `copilot plugin install kling-ai-cn@klingai-cn` for China.
- **[VS Code with Copilot](https://code.visualstudio.com/docs/agent-customization/agent-plugins):** use the matching install link in the table. VS Code asks you to confirm before registering the source and installing the plugin. If the link does not open, run **Chat: Install Plugin from Source** and enter the matching repository URL above.
- **Other MCP clients:** use the matching `/mcp/plugin` endpoint from the table and complete OAuth with the same region's Kling account.

VS Code plugin links require Copilot Agent Host support and the `chat.plugins.enabled` setting. The first connection uses the selected region's OAuth flow.

## Connect from popular MCP coding clients

Use the endpoint for your Kling account region. Complete OAuth with the same regional Kling account when prompted.

### CodeBuddy CLI

```bash
codebuddy mcp add --scope user --transport http kling-ai https://kling.ai/mcp/plugin
```

For China accounts, replace the URL with `https://klingai.com/mcp/plugin`. See the [CodeBuddy MCP guide](https://www.codebuddy.ai/docs/cli/mcp).

### Kimi Code CLI

```bash
kimi mcp add --transport http --auth oauth kling-ai https://kling.ai/mcp/plugin
```

For China accounts, replace the URL with `https://klingai.com/mcp/plugin`. See the [Kimi MCP command reference](https://github.com/MoonshotAI/kimi-cli/blob/main/docs/en/reference/kimi-mcp.md).

### GitHub Copilot CLI

```bash
copilot mcp add --transport http kling-ai https://kling.ai/mcp/plugin
```

For China accounts, replace the URL with `https://klingai.com/mcp/plugin`. See the [Copilot CLI MCP guide](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-mcp-servers).

### OpenCode

Add this entry to `opencode.json`, then run `opencode mcp auth kling-ai` to authorize:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "servers": {
      "kling-ai": {
        "type": "remote",
        "url": "https://kling.ai/mcp/plugin"
      }
    }
  }
}
```

For China accounts, replace the URL with `https://klingai.com/mcp/plugin`. See the [OpenCode MCP guide](https://opencode.ai/docs/mcp-servers/).

## Connect from Manus

In Manus, open **Connectors** → **Create** → **Add MCP by URL**. Add the endpoint for your Kling account region and authorize that account if prompted.

- Global: name the server `Kling AI Global` and use `https://kling.ai/mcp/plugin`.
- China: name the server `Kling AI China` and use `https://klingai.com/mcp/plugin`.

See the [Manus MCP connector guide](https://manus.im/docs/integrations/mcp-connectors) and its [custom MCP setup instructions](https://www.manus.im/features/connectors#custom-mcp-server-and-api-key).

## Install in Hermes Agent

Hermes can search the Kling CLI Skill from the skills.sh catalog:

```bash
hermes skills search kling --source skills-sh
hermes skills install skills-sh/klingai-tech/skills/kling-cli
```

Install the regional Kling CLI package and sign in as described above before using the Skill. See the [Hermes skills guide](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills).

## Other official plugin integrations

- [Cursor](https://github.com/klingai-tech/cursor-plugin)
- [ChatGPT app directory](https://chatgpt.com/plugins/plugins_6a86c3ff68a08191be42add5a9567875) · [source](https://github.com/klingai-dev/chatgpt-plugin)
- [OpenClaw Global on ClawHub](https://hub.openclaw.ai/klingai-dev/plugins/kling-ai-global-openclaw) · [China on ClawHub](https://hub.openclaw.ai/klingai-dev/plugins/kling-ai-openclaw) · [source](https://github.com/klingai-dev/openclaw-plugin)
- [DeepSeek](https://github.com/klingai-dev/deepseek-plugin)
- [Pi](https://github.com/klingai-dev/pi-plugin)
- [Antigravity](https://github.com/klingai-dev/antigravity-plugin)

Generation uses credits from the connected regional Kling account. Check the official regional guide for client-specific setup, supported models, and usage details:

- [Global Kling MCP and CLI guide](https://kling.ai/app/mcp/guide)
- [China Kling MCP and CLI guide](https://klingai.com/app/mcp/guide)
