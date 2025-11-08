# Progressive Discovery Examples

Examples of discovering and using MCP tools, regardless of which servers are available.

## General Principle

**Don't assume which tools exist.** Always discover what's available in the current environment.

## Example 1: First-Time Discovery

### Scenario: You've never explored this system

```bash
# Step 1: Check if progressive discovery exists
ls ~/.mcp-catalogue/
```

**Possible outcomes:**

**A) Directory exists:**
```
coretx/
context7/
index.ts
package.json
...
```
→ Proceed to Step 2

**B) Directory doesn't exist:**
```
ls: ~/.mcp-catalogue/: No such file or directory
```
→ Tell user: "Progressive MCP discovery isn't set up. Would you like me to implement it using the progressive-mcp-discovery skill?"

### Step 2: Discover available servers

```bash
cd ~/.mcp-catalogue && pnpm run discover
```

**Example output:**
```
Available MCP Servers:

  server-a (5 tools)
  server-b (2 tools)
  server-c (0 tools)

Usage: npm run discover -- list <server-name>
```

**Analysis**:
- 2 active servers (server-a, server-b)
- server-c not yet implemented
- Don't assume what these servers do

### Step 3: Explore each server

```bash
# Explore server-a
cd ~/.mcp-catalogue && pnpm run discover -- list server-a

# Explore server-b
cd ~/.mcp-catalogue && pnpm run discover -- list server-b
```

**Now you know**:
- What tools each server provides
- Brief description of each tool
- Still haven't loaded any implementations

## Example 2: User Asks Generic Question

### Scenario: "What capabilities do you have?"

```bash
# Step 1: List servers
cd ~/.mcp-catalogue && pnpm run discover
```

**Response to user:**
"I have access to MCP tools organized in servers. Let me check what's available..."

```bash
# Step 2: List tools in each active server
for server in $(cd ~/.mcp-catalogue && pnpm run discover 2>/dev/null | grep '(' | awk '{print $1}'); do
  echo "=== $server ==="
  cd ~/.mcp-catalogue && pnpm run discover -- list $server
done
```

**Response format:**
"Here's what I can do:

**Server A** (5 tools):
- tool-1: Brief description
- tool-2: Brief description
...

**Server B** (2 tools):
- tool-x: Brief description
- tool-y: Brief description

Would you like more details on any of these?"

**Tokens used**: Minimal - just CLI output, no implementations loaded.

## Example 3: User Asks About Specific Capability

### Scenario: "Can you look up documentation for libraries?"

```bash
# Step 1: Search for relevant tools
cd ~/.mcp-catalogue && pnpm run discover -- list server-a | grep -i "doc\|librar\|search"
cd ~/.mcp-catalogue && pnpm run discover -- list server-b | grep -i "doc\|librar\|search"
```

**Possible outcomes:**

**A) Found relevant tools:**
```
resolve-library-id
  Resolves a package name to a compatible library ID

get-library-docs
  Fetches up-to-date documentation for a library
```

→ "Yes! I found documentation tools in server-b. Let me get more details..."

**B) No relevant tools:**
```
(no matches)
```

→ "I don't see documentation lookup tools in the available servers. The current capabilities are: [list what's actually available]"

### Step 2: If found, get details

```bash
cd ~/.mcp-catalogue && pnpm run discover -- info server-b resolve-library-id
cd ~/.mcp-catalogue && pnpm run discover -- info server-b get-library-docs
```

### Step 3: Explain workflow

"To look up documentation, I would:
1. Use 'resolve-library-id' to find the library
2. Use 'get-library-docs' to fetch the documentation
Would you like me to proceed?"

## Example 4: Discovering Tool Parameters

### Scenario: Need to understand how to use a specific tool

```bash
# Step 1: Get tool info
cd ~/.mcp-catalogue && pnpm run discover -- info server-a create-thing

# Output shows input type name:
# Input Type: CreateThingInput
```

### Step 2: Look up the type definition

```bash
cat ~/.mcp-catalogue/server-a/types.ts | grep -A 10 "CreateThingInput"
```

**Output:**
```typescript
export interface CreateThingInput {
  name: string;          // Required
  options?: ThingOptions;  // Optional
  metadata?: Record<string, unknown>;  // Optional
}
```

### Step 3: Explain to user

"The create-thing tool accepts:
- name (required): string
- options (optional): ThingOptions
- metadata (optional): any key-value pairs

Would you like me to create a thing? I'll need a name at minimum."

## Example 5: Exploring Unknown Server

### Scenario: Discover a server you've never seen before

```bash
# Step 1: List its tools
cd ~/.mcp-catalogue && pnpm run discover -- list unknown-server
```

**Output:**
```
Tools in unknown-server:

  alpha-tool
    Does something with alpha

  beta-tool
    Processes beta data

  gamma-tool
    Transforms gamma to delta
```

### Step 2: Get details on interesting tools

```bash
cd ~/.mcp-catalogue && pnpm run discover -- info unknown-server alpha-tool
```

### Step 3: Read implementation if needed

```bash
cat ~/.mcp-catalogue/unknown-server/alpha-tool.ts
```

### Pattern:
1. **List** tools (lightweight)
2. **Info** on interesting ones (medium)
3. **Read** file only if actually using (heavier)

## Example 6: Filesystem Exploration (Alternative Method)

### When CLI isn't available or you prefer direct filesystem

```bash
# List all servers (directories)
ls -d ~/.mcp-catalogue/*/ | xargs basename

# List tools in a server (excluding index and types)
ls ~/.mcp-catalogue/server-a/*.ts | grep -v "index\|types" | xargs basename -s .ts

# Quick count
ls ~/.mcp-catalogue/server-a/*.ts | grep -v "index\|types" | wc -l
```

**Pattern**: Filesystem as the source of truth, not hardcoded knowledge.

## Example 7: Multi-Step Workflow Discovery

### Scenario: Tool workflow requires multiple steps

```bash
# Discover all tools in server
cd ~/.mcp-catalogue && pnpm run discover -- list workflow-server
```

**Output:**
```
Tools in workflow-server:

  step-one
    First step: prepare data

  step-two
    Second step: process prepared data

  step-three
    Final step: output results
```

**Pattern Recognition**: Tool names suggest a sequence. Verify by:

```bash
# Check each tool's description and parameters
cd ~/.mcp-catalogue && pnpm run discover -- info workflow-server step-one
cd ~/.mcp-catalogue && pnpm run discover -- info workflow-server step-two
```

**Explain to user**: "This looks like a 3-step workflow: prepare → process → output. Would you like me to walk through it?"

## Example 8: Handling Missing Tools

### Scenario: User asks for capability that doesn't exist

```bash
# Check all servers
cd ~/.mcp-catalogue && pnpm run discover
for server in server-a server-b; do
  cd ~/.mcp-catalogue && pnpm run discover -- list $server | grep -i "email\|mail\|send"
done
```

**No results:**

Response: "I don't see email-related tools in the current MCP servers. Available capabilities are:
- Server A: [high-level summary]
- Server B: [high-level summary]

Would any of these help with your task, or would you like to add an email MCP server?"

## Example 9: Type Exploration for Complex Returns

### Scenario: Understanding what a tool returns

```bash
# Tool file shows return type
cat ~/.mcp-catalogue/server-a/get-complex-data.ts
# Returns: Promise<MCPToolResponse<ComplexDataResponse>>
```

### Step 2: Look up the response type

```bash
cat ~/.mcp-catalogue/server-a/types.ts | grep -A 20 "ComplexDataResponse"
```

**Output:**
```typescript
export interface ComplexDataResponse {
  items: DataItem[];
  metadata: {
    total: number;
    page: number;
  };
  links?: {
    next?: string;
    prev?: string;
  };
}
```

### Step 3: Follow nested types

```bash
cat ~/.mcp-catalogue/server-a/types.ts | grep -A 10 "interface DataItem"
```

**Only do this when**: User asks specific questions about return structure.

## Example 10: Version-Agnostic Discovery

### Scenario: Working on different machines with different tool sets

**Machine A:**
```bash
cd ~/.mcp-catalogue && pnpm run discover
# Shows: coretx, context7, custom-server
```

**Machine B:**
```bash
cd ~/.mcp-catalogue && pnpm run discover
# Shows: different-server, another-server
```

**Pattern**: Always discover what's available, never assume.

```bash
# Generic discovery script works everywhere
cd ~/.mcp-catalogue && pnpm run discover
cd ~/.mcp-catalogue && pnpm run discover -- list $(first-server-found)
```

## Key Takeaways

### 1. Always Discover First
Don't assume tools exist. Check what's available.

### 2. Filesystem is Source of Truth
The directory structure tells you what's real.

### 3. Progressive Loading
- List → lightweight
- Info → medium
- Read file → heavier
- Only load what you need

### 4. Generic Patterns
These discovery patterns work regardless of which servers/tools exist.

### 5. Handle Missing Gracefully
If tools don't exist, tell user what IS available.

### 6. Let User Guide Depth
- Vague question → shallow exploration
- Specific question → deep dive

### 7. Compose Workflows
Discover related tools and explain multi-step workflows.

## Universal Discovery Template

```bash
# 1. Check system exists
ls ~/.mcp-catalogue/ || exit 1

# 2. List servers
cd ~/.mcp-catalogue && pnpm run discover

# 3. For each relevant server:
cd ~/.mcp-catalogue && pnpm run discover -- list <server>

# 4. For specific tools:
cd ~/.mcp-catalogue && pnpm run discover -- info <server> <tool>

# 5. Read implementation if using:
cat ~/.mcp-catalogue/servers/<server>/<tool>.ts

# 6. Check types if needed:
cat ~/.mcp-catalogue/servers/<server>/types.ts | grep -A N "<Type>"
```

This template works for **any** progressive MCP discovery system, regardless of which servers are installed.
