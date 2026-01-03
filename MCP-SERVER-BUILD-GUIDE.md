# How to Build an MCP Server - Step-by-Step Guide

This guide explains how we built a GitHub MCP (Model Context Protocol) server from scratch.

## What is an MCP Server?

An MCP (Model Context Protocol) server is a service that allows AI assistants (like Claude in Cursor) to interact with external systems and data sources. It provides tools that the AI can use to perform actions.

## Step-by-Step Build Process

### Step 1: Project Setup

**What we did:**
- Created a new directory: `github-mcp-server`
- Initialized a Node.js project

**Files created:**
- `package.json` - Project configuration and dependencies

**Key dependencies:**
```json
{
  "@modelcontextprotocol/sdk": "^0.5.0",  // MCP SDK
  "@octokit/rest": "^20.0.2",              // GitHub API client
  "dotenv": "^16.3.1"                      // Environment variables
}
```

**Why:**
- `@modelcontextprotocol/sdk` provides the MCP protocol implementation
- `@octokit/rest` handles GitHub API calls
- `dotenv` loads configuration from `.env` file

---

### Step 2: Create the Main Server File

**What we did:**
- Created `src/index.js` - The main MCP server implementation

**Key components:**

#### 2.1 Import Required Modules
```javascript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { CallToolRequestSchema, ListToolsRequestSchema } from "@modelcontextprotocol/sdk/types.js";
import { Octokit } from "@octokit/rest";
import dotenv from "dotenv";
```

**Why:**
- `Server` - Core MCP server class
- `StdioServerTransport` - Communication via stdin/stdout
- `CallToolRequestSchema` - Schema for tool execution requests
- `ListToolsRequestSchema` - Schema for listing available tools
- `Octokit` - GitHub API client
- `dotenv` - Load environment variables

#### 2.2 Initialize GitHub Client
```javascript
dotenv.config();
const octokit = new Octokit({
  auth: process.env.GITHUB_TOKEN,
});
```

**Why:**
- Loads `.env` file
- Creates authenticated GitHub API client

#### 2.3 Create Server Class
```javascript
class GitHubMCPServer {
  constructor() {
    this.server = new Server({
      name: "github-mcp-server",
      version: "1.0.0",
    }, {
      capabilities: {
        tools: {},
      },
    });
  }
}
```

**Why:**
- Encapsulates all server functionality
- Configures server name and capabilities

---

### Step 3: Define Available Tools

**What we did:**
- Implemented `setupHandlers()` method
- Defined tool schemas using `ListToolsRequestSchema`

**Tool structure:**
```javascript
{
  name: "tool_name",
  description: "What the tool does",
  inputSchema: {
    type: "object",
    properties: {
      param1: { type: "string", description: "..." },
      param2: { type: "number", description: "..." }
    },
    required: ["param1"]
  }
}
```

**Tools we created:**
1. `get_repo_info` - Get repository information
2. `list_issues` - List repository issues
3. `get_issue` - Get specific issue details
4. `list_pull_requests` - List pull requests
5. `get_file_contents` - Read file contents
6. `list_repository_files` - List directory contents
7. `search_code` - Search code in repository
8. `create_or_update_file` - Create or update files

**Why:**
- Tools define what actions the AI can perform
- Schemas validate input parameters
- Descriptions help AI understand tool purpose

---

### Step 4: Implement Tool Handlers

**What we did:**
- Created handler for `CallToolRequestSchema`
- Implemented switch statement to route tool calls
- Created methods for each tool

**Handler structure:**
```javascript
this.server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;
  
  switch (name) {
    case "tool_name":
      return await this.toolMethod(args);
    // ... more cases
  }
});
```

**Example implementation:**
```javascript
async getRepoInfo() {
  const { data } = await octokit.repos.get({
    owner: REPO_OWNER,
    repo: REPO_NAME,
  });
  
  return {
    content: [{
      type: "text",
      text: JSON.stringify(data, null, 2)
    }]
  };
}
```

**Why:**
- Handlers execute the actual work
- Return structured responses
- Handle errors gracefully

---

### Step 5: Configure Environment Variables

**What we did:**
- Created `.env.example` template
- Created `.env` file with actual values

**Required variables:**
```
GITHUB_TOKEN=your_github_token_here
GITHUB_REPO_OWNER=your_username
GITHUB_REPO_NAME=your_repo_name
```

**Why:**
- Keeps sensitive data (tokens) out of code
- Allows easy configuration
- `.env` is in `.gitignore` for security

---

### Step 6: Set Up GitHub Authentication

**What we did:**
1. Created GitHub Personal Access Token
   - Go to: https://github.com/settings/tokens
   - Generate new token (classic)
   - Select `repo` scope (required for write operations)
   - Copy token

2. Added token to `.env` file

**Why:**
- GitHub API requires authentication
- `repo` scope allows read/write access
- Token must have correct permissions

---

### Step 7: Start the Server

**What we did:**
- Implemented `run()` method
- Connected server to stdio transport

```javascript
async run() {
  const transport = new StdioServerTransport();
  await this.server.connect(transport);
  console.error("GitHub MCP server running on stdio");
}
```

**Why:**
- Stdio transport allows communication via stdin/stdout
- This is how Cursor/Claude Desktop connects to MCP servers

---

### Step 8: Configure in Cursor/Claude Desktop

**What we did:**
- Added server configuration to MCP settings

**For Cursor:**
```json
{
  "mcpServers": {
    "github": {
      "command": "node",
      "args": ["C:\\Users\\GadOf\\github-mcp-server\\src\\index.js"],
      "env": {
        "GITHUB_TOKEN": "your_token",
        "GITHUB_REPO_OWNER": "GadOfir",
        "GITHUB_REPO_NAME": "Promts"
      }
    }
  }
}
```

**Why:**
- Tells Cursor how to start and connect to your MCP server
- Provides environment variables
- Enables AI to use your tools

---

## Key Concepts

### 1. MCP Protocol
- **Request/Response**: AI sends requests, server responds
- **Tool Discovery**: AI asks what tools are available
- **Tool Execution**: AI calls tools with parameters
- **Error Handling**: Server returns errors in structured format

### 2. Tool Schema
- Defines what parameters each tool accepts
- Validates input before execution
- Helps AI understand how to use tools

### 3. Response Format
```javascript
{
  content: [{
    type: "text",
    text: "response data"
  }],
  isError: false  // true if error occurred
}
```

### 4. Error Handling
- Try/catch blocks around API calls
- Return structured error responses
- Log errors for debugging

---

## Testing Your MCP Server

### 1. Test Locally
```bash
npm start
```

### 2. Test Individual Tools
- Create test scripts (like we did with `test-create-file.js`)
- Verify GitHub API calls work
- Check token permissions

### 3. Test in Cursor
- Configure MCP server in settings
- Restart Cursor
- Ask AI to use your tools

---

## Common Issues & Solutions

### Issue: Token doesn't have permissions
**Solution:** Create new token with `repo` scope

### Issue: Server won't start
**Solution:** 
- Check Node.js version (need 18+)
- Verify dependencies installed (`npm install`)
- Check for syntax errors

### Issue: Tools not available in Cursor
**Solution:**
- Verify MCP configuration in Cursor settings
- Restart Cursor
- Check server is running (should see in logs)

### Issue: 403 Forbidden errors
**Solution:**
- Token missing `repo` scope
- Repository is private and token lacks access
- Token expired or invalid

---

## What Makes a Good MCP Server?

1. **Clear Tool Names**: Descriptive, action-oriented
2. **Good Descriptions**: Help AI understand tool purpose
3. **Proper Schemas**: Validate inputs, prevent errors
4. **Error Handling**: Graceful failures with helpful messages
5. **Documentation**: README explaining setup and usage
6. **Security**: Never commit tokens, use environment variables

---

## Next Steps

1. **Add More Tools**: Extend functionality as needed
2. **Error Handling**: Improve error messages
3. **Caching**: Cache API responses for performance
4. **Logging**: Add better logging for debugging
5. **Testing**: Write unit tests for tools
6. **Documentation**: Keep README updated

---

## Summary

Building an MCP server involves:
1. ✅ Setting up Node.js project with MCP SDK
2. ✅ Creating server class with tool definitions
3. ✅ Implementing tool handlers
4. ✅ Configuring authentication
5. ✅ Testing tools work correctly
6. ✅ Integrating with Cursor/Claude Desktop

The key is understanding:
- **Tools** = Actions AI can perform
- **Schemas** = Define tool inputs
- **Handlers** = Execute tool logic
- **Transport** = Communication method (stdio)

Once set up, your MCP server extends AI capabilities to interact with external systems!

