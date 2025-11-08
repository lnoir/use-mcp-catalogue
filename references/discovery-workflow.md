# Progressive Discovery Workflow

Detailed step-by-step process for discovering and using MCP tools progressively.

## Core Principle

**DO NOT** load all tool definitions into context. Instead, explore the filesystem progressively, reading only what you need when you need it.

This mimics how developers work with libraries:
1. Check what libraries are available
2. Explore the API surface
3. Read docs for specific functions
4. Use the functions

## Complete Workflow

### Phase 1: System Discovery

#### Step 1.1: Verify System Exists

```bash
ls ~/.mcp-catalogue/
```

**Expected output:**
```
chrome-devtools/
context7/
coretx/
example.ts
index.ts
mcp-client.ts
node_modules/
package.json
pnpm-lock.yaml
README.md
servers.json
tsconfig.json
types.ts
```

**If not found**: Suggest using the `progressive-mcp-discovery` skill to implement the system.

#### Step 1.2: List Available Servers

```bash
cd ~/.mcp-catalogue && pnpm run discover
```

**Expected output:**
```
Available MCP Servers:

  coretx (5 tools)
  context7 (2 tools)
  chrome-devtools (0 tools)

Usage: npm run discover -- list <server-name>
```

**Analysis**: You now know:
- Which servers are available
- How many tools each has
- No tool definitions loaded yet!

### Phase 2: Server Exploration

#### Step 2.1: List Tools in a Server

```bash
cd ~/.mcp-catalogue && pnpm run discover -- list coretx
```

**Expected output:**
```
Tools in coretx:

  create-ai-note
    Create a new AI note in Coretx

  create-session
    Create a new Coretx session

  get-active-session
    Get the currently active Coretx session

  get-ai-notes
    Get AI notes from Coretx

  session-add-entry
    Add an entry to the current Coretx session
```

**Analysis**: You now know:
- Tool names (kebab-case filenames)
- Brief description of each
- Still no full tool definitions loaded!

#### Step 2.2: Alternative - Filesystem Listing

```bash
ls ~/.mcp-catalogue/servers/coretx/*.ts | grep -v "index.ts" | grep -v "types.ts"
```

**Output:**
```
~/.mcp-catalogue/servers/coretx/create-ai-note.ts
~/.mcp-catalogue/servers/coretx/create-session.ts
~/.mcp-catalogue/servers/coretx/get-active-session.ts
~/.mcp-catalogue/servers/coretx/get-ai-notes.ts
~/.mcp-catalogue/servers/coretx/session-add-entry.ts
```

**When to use**: When you want raw filenames without running the CLI.

### Phase 3: Tool Investigation (On-Demand)

#### Step 3.1: Get Tool Details

Only when you need to use a specific tool:

```bash
cd ~/.mcp-catalogue && pnpm run discover -- info coretx create-session
```

**Expected output:**
```
Tool: coretx/create-session

Description:
Create a new Coretx session
 * Starts a new session with an optional name and metadata. The new session
becomes the active session for subsequent operations.

Input Type: CreateSessionInput
See: coretx/types.ts for full type definition

File: /Users/user/.mcp-catalogue/coretx/create-session.ts
```

**Analysis**: You now know:
- What the tool does
- What input type it expects
- Where to find more details

#### Step 3.2: Read Tool Implementation

For exact usage and function signature:

```bash
cat ~/.mcp-catalogue/servers/coretx/create-session.ts
```

**Output:**
```typescript
/**
 * Create a new Coretx session
 *
 * Starts a new session with an optional name and metadata. The new session
 * becomes the active session for subsequent operations.
 */

import { callMCPTool } from '../mcp-client.js';
import type { CreateSessionInput, CreateSessionResponse } from './types.js';
import type { MCPToolResponse } from '../types.js';

export async function createSession(
  input: CreateSessionInput = {}
): Promise<MCPToolResponse<CreateSessionResponse>> {
  return callMCPTool<CreateSessionResponse>(
    'coretx',
    'coretx_create_session',
    input
  );
}
```

**Analysis**:
- Function is `createSession()`
- Takes `CreateSessionInput` (with default empty object)
- Returns wrapped response
- Actual MCP tool name is `coretx_create_session`

#### Step 3.3: Check Type Definitions

To see exact parameters:

```bash
cat ~/.mcp-catalogue/servers/coretx/types.ts | grep -A 10 "CreateSessionInput"
```

**Output:**
```typescript
export interface CreateSessionInput {
  name?: string;
  metadata?: Record<string, unknown>;
}

export interface CreateSessionResponse {
  session: Session;
}
```

**Analysis**:
- Both `name` and `metadata` are optional
- Response contains a `Session` object
- Can now explain to user what parameters are available

#### Step 3.4: Check Related Types (if needed)

```bash
cat ~/.mcp-catalogue/servers/coretx/types.ts | grep -A 10 "interface Session"
```

**Output:**
```typescript
export interface Session {
  id: string;
  name?: string;
  createdAt: string;
  lastActivity: string;
  entries?: SessionEntry[];
  metadata?: Record<string, unknown>;
}
```

**Analysis**: Now you understand the full structure of what's returned.

### Phase 4: Usage Decision

Based on what you've learned, you can now:

1. **Explain to user**: Describe what the tool does and what parameters it accepts
2. **Provide example**: Show how they would call it
3. **Suggest use case**: When this tool would be useful
4. **Compose with other tools**: If needed, discover other related tools

## Example Complete Discovery Session

### Scenario: User asks "Can I create a new session?"

#### Step 1: Check if sessions are supported
```bash
cd ~/.mcp-catalogue && pnpm run discover -- list coretx | grep session
```

**Result**: See `create-session`, `get-active-session`, `session-add-entry`

#### Step 2: Get details on create-session
```bash
cd ~/.mcp-catalogue && pnpm run discover -- info coretx create-session
```

**Result**: Learn it creates sessions with optional name and metadata

#### Step 3: Check parameters
```bash
cat ~/.mcp-catalogue/servers/coretx/types.ts | grep -A 5 "CreateSessionInput"
```

**Result**: See `name?: string` and `metadata?: Record<string, unknown>`

#### Step 4: Respond to user
```
Yes! You can create a new session using the coretx create-session tool.

Parameters (all optional):
- name: string - Give your session a name
- metadata: object - Add custom metadata

Example:
{
  name: "My Project Session",
  metadata: { project: "blackout", purpose: "feature development" }
}

The new session will become your active session automatically.
```

**Tokens used**: Minimal! Only loaded specific files needed.

## Progressive Pattern vs. Traditional Pattern

### Traditional (Loading Everything) ❌

```typescript
// Loads ALL tool definitions into context
import * as coretx from '~/.mcp-catalogue/servers/coretx/index.ts';
import { CreateSessionInput, CreateSessionResponse, ... } from '~/.mcp-catalogue/servers/coretx/types.ts';

// Thousands of tokens consumed just to answer one question
```

### Progressive (On-Demand) ✅

```bash
# 1. Quick discovery (minimal tokens)
pnpm run discover -- list coretx

# 2. Specific info (only when needed)
pnpm run discover -- info coretx create-session

# 3. Type details (only for this tool)
cat types.ts | grep -A 5 "CreateSessionInput"

# Tiny fraction of tokens used
```

## When to Read What

### Read Immediately (Lightweight)
- Server list: `pnpm run discover`
- Tool list: `pnpm run discover -- list <server>`
- Directory contents: `ls ~/.mcp-catalogue/servers/<server>/`

### Read On-Demand (When Relevant)
- Tool details: `pnpm run discover -- info <server> <tool>`
- Tool implementation: `cat <server>/<tool>.ts`
- Specific types: `grep -A N "<Type>" types.ts`

### Avoid Reading (Heavy)
- Entire index files
- All type definitions at once
- Multiple tool files when only need one
- Implementation details unless actually using the tool

## Common Discovery Patterns

### Pattern 1: "What can I do with X?"

```bash
# List capabilities
pnpm run discover -- list <server>

# Scan descriptions
# Identify relevant tools
# Read those specific tools
```

### Pattern 2: "How do I use Y?"

```bash
# Get tool info
pnpm run discover -- info <server> <tool>

# Read implementation
cat <server>/<tool>.ts

# Check types
cat <server>/types.ts | grep -A 10 "<InputType>"
```

### Pattern 3: "What parameters does Z accept?"

```bash
# Quick check
pnpm run discover -- info <server> <tool>

# Detailed check
cat <server>/types.ts | grep -A 20 "<InputType>"
```

### Pattern 4: "What's the difference between A and B?"

```bash
# Compare tool descriptions
pnpm run discover -- info <server> <tool-a>
pnpm run discover -- info <server> <tool-b>

# If needed, read both files
cat <server>/<tool-a>.ts
cat <server>/<tool-b>.ts
```

## Key Takeaways

1. **Always start with discovery CLI** - lightweight and fast
2. **Read only what you need** - don't preemptively load definitions
3. **Use grep for targeted lookups** - extract specific types
4. **Explain capabilities without loading everything** - use tool descriptions
5. **Defer implementation reading** - only when actually using the tool

This progressive approach is what makes the system scalable to dozens or hundreds of tools without context bloat.
