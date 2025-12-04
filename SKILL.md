---
name: use-mcp-tools
description: Discover and use MCP tools from the progressive discovery system at ~/.mcp-catalogue/. Learn what's available through filesystem exploration without loading all tools into context.
allowed-tools: [Read, Bash, Glob]
---

# Use MCP Tools

Discover and use MCP tools progressively from `~/.mcp-catalogue/` without loading all tool definitions into context.

## When to Use This Skill

Use this skill when:
- User asks "What can you do?" or about available MCP capabilities
- User mentions wanting to use MCP tools or features
- You need to explore what MCP servers and tools are available
- User mentions any MCP server name (Coretx, Figma, Postman, Context7, Chrome DevTools, etc.)
- User asks about specific capabilities without knowing which tool provides them

## Quick Start Workflow

**CRITICAL**: Never assume which servers or tools exist. Always discover dynamically.

1. **Check if system exists**: `ls ~/.mcp-catalogue/`
   - If missing, it can be cloned from from https://github.com/lnoir/mcp-catalogue.git.
   - Use the [mcp-catalogue-add](https://github.com/lnoir/mcp-catalogue-add.git) skill to add MCP server tools

2. **Read the system README**: `cat ~/.mcp-catalogue/README.md`
   - Contains complete documentation on architecture, available servers, and CLI usage
   - ~1,500 tokens but covers everything you need

3. **Discover available servers**: `cd ~/.mcp-catalogue && pnpm run discover`
   - Lists all servers with tool counts without loading definitions

4. **Explore specific tools as needed**:
   - List server's tools: `pnpm run discover -- list <server>`
   - Get tool details: `pnpm run discover -- info <server> <tool>`
   - Read implementation: `cat ~/.mcp-catalogue/servers/<server>/<tool>.ts`
   - Check types: `cat ~/.mcp-catalogue/servers/<server>/types.ts`

## How to Call MCP Tools

There are two ways to call MCP tools depending on whether they need persistent state:

### 1. One-Off Tool Calls (Stateless)

For most tools that don't need persistent connections:

```bash
cd ~/.mcp-catalogue
pnpm run call <server> <tool> [params]
```

**Params can be**:
- JSON string: `'{"key":"value"}'`
- File: `@params.json`
- Stdin: `-` (read from pipe)
- Omitted: `{}` (empty params)

**Examples:**
```bash
# Get a Jira issue
pnpm run call atlassian getJiraIssue '{"cloudId":"https://site.atlassian.net","issueIdOrKey":"API-86"}'

# Search Jira issues with JQL
pnpm run call atlassian searchJiraIssuesUsingJql '{"cloudId":"https://site.atlassian.net","jql":"project=API AND status=\"To Do\""}'

# Get AI notes from Coretx
pnpm run call coretx getAiNotes '{"category":"","tags":"","limit":5,"offset":0}'

# Create issue from JSON file
pnpm run call atlassian createJiraIssue @create-issue-params.json

# Pipe params from stdin
echo '{"cloudId":"...","issueIdOrKey":"API-86"}' | pnpm run call atlassian getJiraIssue -
```

### 2. Persistent Sessions (Stateful)

For servers that need persistent state (browsers, databases, long-running processes):

```bash
# Start a session
pnpm run session -- start <server>

# Call tools on the session (multiple times)
pnpm run session -- call <server> <tool> '{...}'

# Stop the session when done
pnpm run session -- stop <server>
```

**Examples:**
```bash
# Start browser session
pnpm run session -- start chrome-devtools

# Navigate and interact
pnpm run session -- call chrome-devtools navigate_page '{"url":"https://example.com"}'
pnpm run session -- call chrome-devtools take_screenshot '{}'
pnpm run session -- call chrome-devtools click_element '{"selector":"button.submit"}'

# Stop when done
pnpm run session -- stop chrome-devtools
```

**When to use sessions:**
- Browser automation (chrome-devtools)
- Database connections
- Any tool that maintains state between calls
- Long-running processes

### Programmatic Usage (Advanced)

If you need TypeScript code with type safety, you can import tools directly:

```typescript
import { initializeServer, closeAllServers } from '~/.mcp-catalogue/mcp-client.js';
import { getJiraIssue } from '~/.mcp-catalogue/servers/atlassian/index.js';
import serverConfig from '~/.mcp-catalogue/servers.json' with { type: 'json' };

async function example() {
  await initializeServer('atlassian', serverConfig.atlassian);
  const result = await getJiraIssue({
    cloudId: 'https://site.atlassian.net',
    issueIdOrKey: 'API-86'
  });
  await closeAllServers();
}
```

See `~/.mcp-catalogue/example.ts` for complete TypeScript examples.

## Key Principles

- **The filesystem is the source of truth** - Don't maintain static lists, explore dynamically
- **Progressive discovery** - Load only what you need, when you need it
- **Never import index files** - They load all tools at once (defeats the purpose)
- **Always start fresh** - Run `pnpm run discover` at the start of each conversation

## Anti-Pattern: Don't Do This

```typescript
// ✗ WRONG - loads all tool definitions into context
import * as coretx from '~/.mcp-catalogue/servers/coretx/index.ts';
import * as context7 from '~/.mcp-catalogue/servers/context7/index.ts';
```

## References

- **[~/.mcp-catalogue/README.md](~/.mcp-catalogue/README.md)**: Complete system documentation (source of truth)
- **[references/usage-patterns.md](references/usage-patterns.md)**: Common patterns and best practices
- **[examples/discovery-examples.md](examples/discovery-examples.md)**: Concrete usage examples
