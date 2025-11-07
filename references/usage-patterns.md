# MCP Tool Usage Patterns

Common patterns and best practices for using MCP tools from `~/.mcp-servers/`.

## Pattern 1: Progressive Discovery

### Anti-Pattern ❌
```typescript
// Loading everything upfront
import * as coretx from '~/.mcp-servers/servers/coretx/index.ts';
import * as context7 from '~/.mcp-servers/servers/context7/index.ts';

// Thousands of tokens consumed before even knowing what user wants
```

### Correct Pattern ✅
```bash
# 1. User asks about capability
# 2. Check what's available
cd ~/.mcp-servers && pnpm run discover

# 3. Explore relevant server
cd ~/.mcp-servers && pnpm run discover -- list coretx

# 4. Get specific tool info
cd ~/.mcp-servers && pnpm run discover -- info coretx create-session

# 5. Read tool file only if using it
cat ~/.mcp-servers/servers/coretx/create-session.ts

# Minimal tokens used, maximum flexibility
```

## Pattern 2: Chained Tool Usage

### Example: Documentation Lookup

Context7 requires two steps: resolve ID, then fetch docs.

```bash
# Step 1: Discover the workflow
pnpm run discover -- list context7
# Shows: resolve-library-id, get-library-docs

# Step 2: Check first tool
pnpm run discover -- info context7 resolve-library-id
# Learn: Takes libraryName, returns libraryId

# Step 3: Check second tool
pnpm run discover -- info context7 get-library-docs
# Learn: Takes context7CompatibleLibraryID (from step 1)

# Step 4: Explain workflow to user
# "First I'll resolve 'react' to get the library ID,
#  then fetch the documentation for hooks"
```

**Best Practice**: Explain the multi-step workflow to the user before executing.

## Pattern 3: Optional Parameters

Many tools have optional parameters. Check types to see what's available.

### Example: create-session

```bash
# Check parameters
cat ~/.mcp-servers/servers/coretx/types.ts | grep -A 5 "CreateSessionInput"

# Output shows:
# name?: string
# metadata?: Record<string, unknown>
```

**Pattern**:
1. Tell user: "Both parameters are optional"
2. Suggest use cases:
   - With name: For named sessions
   - With metadata: For custom tracking
   - Without either: Quick unnamed session

## Pattern 4: Type Exploration

When you need to understand complex types:

```bash
# 1. Find the type name from tool file
cat ~/.mcp-servers/servers/coretx/get-active-session.ts
# Shows: GetActiveSessionResponse

# 2. Look up the type
cat ~/.mcp-servers/servers/coretx/types.ts | grep -A 20 "GetActiveSessionResponse"
# Shows: { session: Session | null }

# 3. Follow nested types
cat ~/.mcp-servers/servers/coretx/types.ts | grep -A 15 "interface Session"
# Shows: Full Session structure
```

**When to do this**: Only when user asks specific questions about return values or structure.

## Pattern 5: Tool Grouping

Group related tools by capability:

### Coretx Tool Groups

**Session Management**:
- `create-session` - Start new session
- `get-active-session` - Get current session
- `session-add-entry` - Log to session

**Note Management**:
- `create-ai-note` - Save note
- `get-ai-notes` - Retrieve notes

**Pattern**: Discover related tools together, explain as a cohesive feature set.

```bash
# Discover all session-related tools
pnpm run discover -- list coretx | grep session

# Output:
# create-session
# get-active-session
# session-add-entry
```

## Pattern 6: Error Handling

Tools may fail for various reasons. Common scenarios:

### Scenario 1: Server not running
```bash
# Check if server is configured
cat ~/.mcp-servers/servers.json | grep -A 5 "coretx"
```

If server needs to be running locally and isn't:
- Inform user
- Explain how to start the server
- Don't attempt to use tools

### Scenario 2: Invalid parameters
```bash
# Always check required vs optional parameters
cat ~/.mcp-servers/servers/<server>/types.ts | grep -A 10 "<InputType>"
```

Before calling a tool:
- Verify all required parameters are provided
- Validate optional parameters are appropriate
- Explain what the tool expects

### Scenario 3: Tool doesn't exist
```bash
# Verify tool exists
ls ~/.mcp-servers/servers/<server>/<tool>.ts
```

If tool not found:
- List available tools: `pnpm run discover -- list <server>`
- Suggest closest match
- Ask user to clarify

## Pattern 7: Explaining Capabilities

When user asks "what can you do?":

```bash
# 1. Quick overview
cd ~/.mcp-servers && pnpm run discover

# 2. For each server, give high-level summary:
# "Coretx (5 tools): Session management and note-taking"
# "Context7 (2 tools): Library documentation lookup"

# 3. If user interested in specific server:
cd ~/.mcp-servers && pnpm run discover -- list coretx

# 4. Explain each tool's purpose without loading implementation
```

**Don't**: Read all tool files to explain capabilities
**Do**: Use discovery CLI and tool descriptions

## Pattern 8: Targeted Type Lookup

Use grep for surgical type extraction:

```bash
# Find a specific interface
cat ~/.mcp-servers/servers/coretx/types.ts | grep -A 10 "CreateSessionInput"

# Find all interfaces with "Session" in name
cat ~/.mcp-servers/servers/coretx/types.ts | grep "interface.*Session"

# Find an interface and its dependencies
cat ~/.mcp-servers/servers/coretx/types.ts | grep -A 30 "interface Session"
```

**Best Practice**: Only grep for the specific types you need, not all types.

## Pattern 9: Documentation Workflow

For Context7 documentation lookups:

### Complete Workflow

```bash
# Step 1: User asks about library
# User: "How do React hooks work?"

# Step 2: Discover Context7 tools (if not already known)
pnpm run discover -- list context7

# Step 3: Explain the process
# "I'll first resolve 'react' to get its library ID,
#  then fetch documentation focused on hooks"

# Step 4: Check tool parameters (first time only)
cat ~/.mcp-servers/servers/context7/types.ts | grep -A 5 "ResolveLibraryIdInput"
cat ~/.mcp-servers/servers/context7/types.ts | grep -A 5 "GetLibraryDocsInput"

# Step 5: Explain what you'll pass
# "Calling with: { libraryName: 'react' }"
# "Then: { context7CompatibleLibraryID: '<from-step-1>', topic: 'hooks', tokens: 3000 }"

# Step 6: Return results to user
```

## Pattern 10: Session Lifecycle

Common session workflow:

```bash
# 1. Create session at start of work
# Tool: create-session
# Parameters: { name: "Feature X Development", metadata: { project: "blackout" } }

# 2. Log important events during work
# Tool: session-add-entry
# Parameters: { entry: "Implemented caret tracking", type: "decision" }

# 3. Review session at end
# Tool: get-active-session
# Parameters: { includeEntries: true }
```

**Best Practice**: Suggest this workflow when user starts a significant task.

## Pattern 11: Note Organization

Effective note-taking with Coretx:

```bash
# Create notes with tags for organization
# Tool: create-ai-note
# Parameters: {
#   content: "Progressive MCP discovery reduces tokens by 98%",
#   tags: ["architecture", "mcp", "optimization"]
# }

# Retrieve notes by query
# Tool: get-ai-notes
# Parameters: { query: "mcp", limit: 10 }
```

**Best Practice**: Encourage using tags for better organization.

## Pattern 12: Filesystem Shortcuts

Quick exploration without CLI:

```bash
# See all Coretx tools
ls ~/.mcp-servers/servers/coretx/*.ts | grep -v "index\|types"

# Count tools
ls ~/.mcp-servers/servers/coretx/*.ts | grep -v "index\|types" | wc -l

# Quick scan of tool names
ls ~/.mcp-servers/servers/coretx/*.ts | grep -v "index\|types" | xargs basename -s .ts

# Find tools matching pattern
ls ~/.mcp-servers/servers/coretx/*session*.ts
```

**When to use**: Quick checks, automation, or when CLI is unavailable.

## Anti-Patterns to Avoid

### Anti-Pattern 1: Premature Loading
❌ Reading all tool files before knowing what user wants
✅ Discover capabilities first, load on-demand

### Anti-Pattern 2: Ignoring Discovery CLI
❌ Manually parsing files to find tools
✅ Use `pnpm run discover` for structured discovery

### Anti-Pattern 3: Hardcoding Tool Knowledge
❌ Assuming you know all available tools
✅ Always check current state with discovery

### Anti-Pattern 4: Loading Index Files
❌ `import * from './coretx/index.ts'`
✅ Read specific tool files only when needed

### Anti-Pattern 5: Skipping Type Checks
❌ Guessing at parameter types
✅ Always check types.ts for accurate info

## Best Practices Summary

### DO:
✓ Start with discovery CLI
✓ Read only specific tools you need
✓ Use grep for targeted type lookup
✓ Explain capabilities without loading implementations
✓ Check types before using tools
✓ Group related tools logically
✓ Validate parameters before calling tools

### DON'T:
✗ Load all tools upfront
✗ Import index files
✗ Read files you don't need
✗ Guess at tool parameters
✗ Skip discovery phase
✗ Assume tools exist without checking

## Token Efficiency Comparison

### Scenario: User asks "Can I create a session?"

**Traditional Approach (All Loaded):**
- Load all Coretx tools: ~500 tokens
- Load all types: ~300 tokens
- Load MCP client: ~200 tokens
- Total: ~1000 tokens

**Progressive Approach:**
- Discovery CLI output: ~50 tokens
- One tool file: ~30 tokens
- Type snippet (grep): ~20 tokens
- Total: ~100 tokens

**Savings: 90%**

This is why progressive discovery scales to dozens of servers with hundreds of tools.

## Contextual Intelligence

### When to go deep:
- User asks specific questions about a tool
- About to actually use a tool
- User wants to understand tool behavior
- Debugging an issue

### When to stay shallow:
- User asks general "what can you do"
- Exploring capabilities
- User hasn't decided what they want yet
- Just need to know if something is possible

**Principle**: Match depth of exploration to user's level of specificity.

## Composing Multiple Tools

Some tasks require multiple tools. Pattern:

1. **Discover** all needed tools
2. **Explain** the workflow to user
3. **Validate** you have all required parameters
4. **Execute** in correct sequence
5. **Report** results

Example: "To set up your session with documentation lookup:
1. create-session (start tracking)
2. resolve-library-id (find React)
3. get-library-docs (fetch hooks docs)
4. session-add-entry (log what you learned)"

## Summary

The key to effective MCP tool usage is **progressive discovery**:
1. Explore lightweight metadata first
2. Drill down only as needed
3. Load implementations only when actually using
4. Use discovery CLI as your primary interface
5. Keep token usage minimal

This pattern allows the system to scale infinitely without context bloat.
