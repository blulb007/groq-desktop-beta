# AGENTS.md - LLM Agent Guide for Groq Desktop

This document provides comprehensive guidance for LLM agents (AI assistants) working on the Groq Desktop codebase. This guide is platform-agnostic and designed for any LLM agent, including Claude, GPT, Llama, Gemini, and others. Last updated: 2025-12-10

## Project Overview

**Groq Desktop** is an Electron-based desktop application that provides a chat interface for Groq's LLM API with advanced features including:
- Multi-platform support (Windows, macOS, Linux)
- MCP (Model Context Protocol) server integration for function calling
- Two API modes: Standard Chat Completions and Responses API (Beta/Agentic)
- Google OAuth integration for Google Connectors (Gmail, Calendar, Drive)
- Vision support for image-capable models (e.g., llama-4)
- Built-in tools for gpt-oss models (Code Interpreter, Browser Search)
- Persistent chat history with auto-generated titles
- Real-time streaming responses with reasoning display
- Tool approval system with configurable permissions

## Tech Stack

### Core Technologies
- **Electron 37.0.0** - Desktop application framework
- **React 19.0.1** - UI library
- **Vite 6.2.6** - Build tool and dev server
- **Tailwind CSS 3.3.3** - Styling framework
- **pnpm 10.9.0** - Package manager

### Key Dependencies
- **groq-sdk 0.16.0** - Groq API client
- **@modelcontextprotocol/sdk 1.7.0** - MCP server support
- **react-router-dom 7.3.0** - Routing
- **react-markdown 10.1.0** - Markdown rendering
- **electron-json-storage 4.6.0** - Persistent storage
- **zod 3.24.2** - Schema validation

### UI Components
- **@radix-ui** - Accessible component primitives (Select, Slot)
- **lucide-react** - Icon library
- **class-variance-authority** - Component variant management
- **next-themes** - Theme management

## Architecture Overview

### Process Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                       Electron App                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌────────────────────┐         ┌─────────────────────┐   │
│  │   Main Process     │ ◄─IPC──►│ Renderer Process    │   │
│  │   (Node.js)        │         │ (Chromium + React)  │   │
│  │                    │         │                     │   │
│  │ - electron/main.js │         │ - src/renderer/     │   │
│  │ - MCP Manager      │         │ - React Components  │   │
│  │ - Chat Handler     │         │ - ChatContext       │   │
│  │ - Tool Execution   │         │ - UI State          │   │
│  │ - OAuth Flows      │         │                     │   │
│  │ - File System      │         │                     │   │
│  └────────┬───────────┘         └─────────────────────┘   │
│           │                                                │
│           │ Bridge: electron/preload.js                    │
│           │ (contextBridge.exposeInMainWorld)              │
│           │                                                │
└───────────┼────────────────────────────────────────────────┘
            │
            ▼
    ┌───────────────────┐
    │   External APIs   │
    │                   │
    │ - Groq API        │
    │ - MCP Servers     │
    │ - Google OAuth    │
    └───────────────────┘
```

### Directory Structure

```
groq-desktop-beta-windows/
├── electron/                    # Main Process (Node.js Backend)
│   ├── main.js                 # App entry, window management, IPC setup
│   ├── preload.js              # Context bridge (security boundary)
│   ├── chatHandler.js          # Groq API communication (both modes)
│   ├── mcpManager.js           # MCP server lifecycle management
│   ├── toolHandler.js          # Tool execution via MCP clients
│   ├── authManager.js          # MCP OAuth 2.0 flows
│   ├── googleOAuthManager.js   # Google OAuth refresh tokens
│   ├── chatHistoryManager.js   # Chat persistence to disk
│   ├── settingsManager.js      # Settings persistence
│   ├── commandResolver.js      # Platform-aware command resolution
│   ├── contextCapture.js       # Global hotkey for context capture
│   ├── messageUtils.js         # Message pruning/cleaning
│   ├── popupWindow.js          # Popup window management
│   ├── windowManager.js        # Window creation utilities
│   ├── utils.js                # Shared utilities
│   └── scripts/                # Platform-specific runners
│       ├── run-node.{sh,cmd,ps1}
│       ├── run-npx.{sh,cmd,ps1}
│       ├── run-uvx.{sh,cmd,ps1}
│       └── run-deno.{sh,cmd,ps1}
│
├── src/renderer/               # Renderer Process (React Frontend)
│   ├── main.jsx               # React entry point
│   ├── App.jsx                # Main app component (chat orchestration)
│   ├── index.css              # Global styles, CSS variables
│   │
│   ├── context/
│   │   └── ChatContext.jsx    # Chat state management (messages, history)
│   │
│   ├── pages/
│   │   ├── Settings.jsx       # Settings UI (API keys, MCP, OAuth)
│   │   └── PopupPage.jsx      # Context capture popup
│   │
│   ├── components/
│   │   ├── ChatInput.jsx      # Message input with image support
│   │   ├── MessageList.jsx    # Conversation display
│   │   ├── Message.jsx        # Individual message rendering
│   │   ├── ToolCall.jsx       # Tool call visualization
│   │   ├── ToolsPanel.jsx     # MCP server/tool management
│   │   ├── ChatHistorySidebar.jsx  # Chat history navigation
│   │   ├── ToolApprovalModal.jsx   # Tool permission requests
│   │   ├── LogViewerModal.jsx      # API request log viewer
│   │   ├── MarkdownRenderer.jsx    # Markdown with syntax highlighting
│   │   └── ui/                # Reusable UI components (shadcn-style)
│   │       ├── button.jsx
│   │       ├── input.jsx
│   │       ├── textarea.jsx
│   │       ├── select.jsx
│   │       ├── card.jsx
│   │       ├── badge.jsx
│   │       ├── label.jsx
│   │       ├── Switch.jsx
│   │       ├── SearchableSelect.jsx
│   │       └── text-shimmer.jsx
│   │
│   └── lib/
│       └── utils.js           # UI utilities (cn, clsx+twMerge)
│
├── shared/                     # Shared between processes
│   └── models.js              # Model configurations and API fetching
│
├── public/                     # Static assets
│   ├── icon.png               # App icon (24488 bytes)
│   └── groqLogo.png           # Groq logo (35314 bytes)
│
├── dist/                       # Vite build output (gitignored)
├── release/                    # Electron build output (gitignored)
│
├── package.json               # Dependencies and scripts
├── vite.config.cjs            # Vite configuration
├── electron-builder.yml       # Electron Builder settings
├── tailwind.config.cjs        # Tailwind CSS configuration
├── postcss.config.cjs         # PostCSS configuration
├── eslint.config.js           # ESLint configuration (Flat config)
└── pnpm-workspace.yaml        # pnpm workspace configuration
```

## UI Files Quick Reference

All custom UI components and styling are located in the following directories:

### Main UI Directory
**Location:** `src/renderer/`

### Core UI Files
- **Entry Point:** `src/renderer/main.jsx`
- **Main App:** `src/renderer/App.jsx`
- **Global Styles:** `src/renderer/index.css`

### State Management
- **Chat Context:** `src/renderer/context/ChatContext.jsx`

### Pages
- **Settings:** `src/renderer/pages/Settings.jsx`
- **Popup:** `src/renderer/pages/PopupPage.jsx`

### Main Components (src/renderer/components/)
| File | Purpose |
|------|---------|
| `ChatInput.jsx` | Message input with image upload |
| `MessageList.jsx` | Conversation message container |
| `Message.jsx` | Individual message rendering |
| `ToolCall.jsx` | Tool call display |
| `ToolsPanel.jsx` | MCP server/tool management panel |
| `ChatHistorySidebar.jsx` | Chat history sidebar |
| `ToolApprovalModal.jsx` | Tool approval dialog |
| `LogViewerModal.jsx` | API request log viewer |
| `MarkdownRenderer.jsx` | Markdown rendering with syntax highlighting |

### UI Primitives (src/renderer/components/ui/)
| File | Purpose |
|------|---------|
| `button.jsx` | Button component |
| `input.jsx` | Input field |
| `textarea.jsx` | Text area |
| `select.jsx` | Select dropdown |
| `card.jsx` | Card container |
| `badge.jsx` | Badge/pill component |
| `label.jsx` | Form label |
| `Switch.jsx` | Toggle switch |
| `SearchableSelect.jsx` | Searchable dropdown |
| `text-shimmer.jsx` | Animated text shimmer effect |

### Styling Configuration
- **Tailwind Config:** `tailwind.config.cjs`
- **PostCSS Config:** `postcss.config.cjs`

### Static Assets
- **Icons/Images:** `public/` directory

## Key Subsystems

### 1. IPC Communication Pattern

**All communication between renderer (frontend) and main process (backend) goes through the preload bridge:**

```javascript
// electron/preload.js - Define API surface
const { contextBridge, ipcRenderer } = require('electron');

contextBridge.exposeInMainWorld('electron', {
  // Example: Send message and get response
  sendMessage: (message) => ipcRenderer.invoke('send-message', message),

  // Example: Start streaming
  startChatStream: (messages, model) => {
    const eventId = Date.now();
    return {
      onContent: (callback) => ipcRenderer.on(`chat-content-${eventId}`, callback),
      onComplete: (callback) => ipcRenderer.on(`chat-complete-${eventId}`, callback),
      cleanup: () => {
        ipcRenderer.removeAllListeners(`chat-content-${eventId}`);
        ipcRenderer.removeAllListeners(`chat-complete-${eventId}`);
      }
    };
  }
});

// electron/main.js - Handle requests
ipcMain.handle('send-message', async (event, message) => {
  const result = await processMessage(message);
  return result;
});

// src/renderer/App.jsx - Use in React
const handleSend = async (message) => {
  const result = await window.electron.sendMessage(message);
  // Handle result
};
```

**Critical Rule:** Never use `require()` or Node.js APIs directly in renderer code. Always go through `window.electron.*` APIs.

### 2. Chat Handler (Two API Modes)

Located in `electron/chatHandler.js`, this is the core of the application's AI interaction.

#### Standard Chat Completions API

```javascript
// Used for: Normal chat, local MCP tools, basic streaming
const stream = await groq.chat.completions.create({
  messages: conversationHistory,
  model: selectedModel,
  tools: mcpTools,  // Local tools only
  stream: true,
  temperature: 0.7
});

for await (const chunk of stream) {
  // Handle content, tool_calls, reasoning
}
```

#### Responses API (Beta/Agentic Mode)

```javascript
// Used for: Remote MCP servers, Google Connectors, advanced agentic workflows
const response = await fetch(`${baseUrl}/openai/v1/chat/responses`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    model: selectedModel,
    messages: conversationHistory,
    tools: allTools,  // Local + remote tools
    mcp_servers: remoteMcpServers,  // SSE/HTTP MCP servers
    google_connectors: googleConnectorConfigs,
    stream: true
  })
});

// Parse Server-Sent Events
const reader = response.body;
// Handle: content, reasoning, tool_calls, pre_calculated_tool_responses, mcp_approval_request
```

**Key Differences:**
- Standard API: Client-side tool execution only
- Responses API: Server-side tool execution for remote MCPs, approval workflow
- Responses API supports `pre_calculated_tool_responses` (server already executed tool)
- Responses API supports `mcp_approval_request`/`mcp_approval_response` flow

### 3. MCP (Model Context Protocol) Server Management

Located in `electron/mcpManager.js`

#### Transport Types Supported

1. **STDIO** - Local processes (command-line tools)
```json
{
  "type": "stdio",
  "command": "node",
  "args": ["server.js"],
  "env": { "API_KEY": "..." }
}
```

2. **SSE** - Server-Sent Events (remote HTTP servers)
```json
{
  "type": "sse",
  "url": "https://example.com/mcp",
  "headers": { "Authorization": "Bearer token" }
}
```

3. **StreamableHTTP** - HTTP streaming (remote servers)
```json
{
  "type": "streamableHttp",
  "url": "https://example.com/mcp",
  "headers": { "X-API-Key": "..." }
}
```

#### Connection Lifecycle

```javascript
// 1. Initialize on app startup
await mcpManager.initialize();

// 2. Auto-connect all servers (1 second delay)
setTimeout(() => mcpManager.autoConnectServers(), 1000);

// 3. Health checks every 60 seconds
setInterval(() => {
  for (const serverId of connectedServers) {
    const isHealthy = await client.listTools();
    if (!isHealthy) disconnectServer(serverId);
  }
}, 60000);

// 4. Tool discovery
const tools = await client.listTools();
discoveredTools.push(...tools.map(t => ({ ...t, serverId })));

// 5. Notify renderer
mainWindow.webContents.send('mcp-status-changed', { serverId, status: 'connected' });
```

### 4. Tool Execution Flow

```
User Message
    ↓
Model Response (with tool_calls)
    ↓
Check Approval Status (localStorage)
    ↓
    ├─ Auto-approved → Execute
    └─ Not approved → Show ToolApprovalModal
            ↓
        User Decision
            ↓
            ├─ Approve Once → Execute
            ├─ Approve Always → Save to localStorage → Execute
            └─ Deny → Add error message
                ↓
        Execute Tool (toolHandler.js)
            ↓
        Find MCP Client by serverId
            ↓
        client.callTool({ name, arguments })
            ↓
        Limit Output Length (20,000 chars)
            ↓
        Add Tool Response to Messages
            ↓
        Continue Conversation (send back to model)
```

**Tool Approval Modes:**
- `prompt` - Ask for each tool (default)
- `always` - Per-tool auto-approval (stored: `tool_approval_${toolName}`)
- `yolo` - All tools auto-approved (stored: `tool_approval_yolo_mode`)

### 5. Authentication Systems

#### Google OAuth (googleOAuthManager.js)

```javascript
// Stores: refresh_token, client_id, client_secret
// Auto-refresh: 5 minutes before expiry
// Usage: Google Connectors in Responses API

const tokenInfo = await getTokenStatus();
// { hasAuth: true, expiresAt: '2025-12-10T10:30:00Z', isValid: true }

// Refresh flow
const newTokens = await refreshAccessToken();
// { access_token, expires_in, refresh_token }
```

#### MCP OAuth (authManager.js)

```javascript
// OAuth 2.0 Authorization Code Flow with PKCE
// Dynamic client registration
// Local callback server (port 10000+)

const result = await handleOAuthFlow(serverUrl, serverId);
// 1. Start local server
// 2. Discover OAuth metadata (/.well-known/oauth-authorization-server)
// 3. Register client (if needed) with redirect_uri
// 4. Open browser for authorization
// 5. Capture callback, exchange code for tokens
// 6. Store tokens, retry MCP connection
// 7. Cleanup server
```

#### API Key Authentication

```javascript
// Stored in: userData/settings.json
// Key: GROQ_API_KEY
// Loaded on startup, used in all API requests

const settings = await settingsManager.getSettings();
const apiKey = settings.GROQ_API_KEY || process.env.GROQ_API_KEY;
```

### 6. Chat History Management

Located in `electron/chatHistoryManager.js`

```javascript
// Storage: userData/chat-history/{chatId}.json

// Structure
{
  id: "uuid-v4",
  title: "Generated by llama-3.1-8b-instant",
  createdAt: "2025-12-09T12:00:00Z",
  updatedAt: "2025-12-09T12:05:00Z",
  model: "llama-3.3-70b-versatile",
  useResponsesApi: false,
  messages: [
    { role: "user", content: "..." },
    { role: "assistant", content: "...", tool_calls: [...] },
    { role: "tool", tool_call_id: "...", content: "..." }
  ]
}

// Auto-save on every message update
// Title generation: After first user message
// Message cleaning: Removes transient properties (isStreaming, liveReasoning)
```

### 7. State Management Pattern

#### Global State (ChatContext)

```jsx
// src/renderer/context/ChatContext.jsx
const ChatContext = createContext();

export function ChatProvider({ children }) {
  const [messages, setMessages] = useState([]);
  const [currentChatId, setCurrentChatId] = useState(null);
  const [chatList, setChatList] = useState([]);

  // Auto-save wrapper
  const setMessagesWithSave = useCallback((newMessages) => {
    setMessages(newMessages);
    if (currentChatIdRef.current) {
      window.electron.updateChatMessages(currentChatIdRef.current, newMessages);
      // Update local chat list timestamp (avoid reload)
      setChatList(prev => prev.map(chat =>
        chat.id === currentChatIdRef.current
          ? { ...chat, updatedAt: new Date().toISOString() }
          : chat
      ));
    }
  }, []);

  return (
    <ChatContext.Provider value={{ messages, setMessagesWithSave, ... }}>
      {children}
    </ChatContext.Provider>
  );
}
```

#### Local Component State (App.jsx)

```jsx
// Model selection, streaming state, UI state
const [selectedModel, setSelectedModel] = useState('llama-3.3-70b-versatile');
const [isLoading, setIsLoading] = useState(false);
const [mcpTools, setMcpTools] = useState([]);
const [showToolApproval, setShowToolApproval] = useState(false);
const [currentToolCall, setCurrentToolCall] = useState(null);
```

### 8. Model Configuration

Located in `shared/models.js`

```javascript
// Dynamic model fetching from Groq API
const models = await getModelsFromAPIWithCache(apiKey);

// 5-minute cache
// Filters: Excludes whisper, guard models; includes only active chat models
// Heuristics:
//   - 'gpt-oss' in name → builtin_tools_supported: true
//   - 'llama-4' in name → vision_supported: true
// Custom models: Merged from settings.customModels

// Structure
{
  "llama-3.3-70b-versatile": {
    context: 32768,
    vision_supported: false,
    builtin_tools_supported: false
  },
  "llama-4-scout-beta-preview": {
    context: 8192,
    vision_supported: true,
    builtin_tools_supported: false
  },
  "gpt-oss-llama-3.1-405b": {
    context: 16384,
    vision_supported: false,
    builtin_tools_supported: true  // Code Interpreter, Browser Search
  }
}
```

## Development Workflows

### Setup

```bash
# 1. Install pnpm globally (if not already installed)
npm install -g pnpm@10.9.0

# 2. Clone repository
git clone <repo-url>
cd groq-desktop-beta-windows

# 3. Install dependencies
pnpm install

# 4. (If pnpm blocks scripts) Approve build scripts
pnpm approve-builds
# Select: electron, esbuild

# 5. Start development server
pnpm dev
```

### Development Commands

```bash
# Development
pnpm dev              # Start Vite + Electron (recommended)
pnpm dev:vite         # Vite dev server only (http://localhost:5173)
pnpm dev:electron     # Electron only (requires built files)

# Building
pnpm build            # Build renderer (Vite → dist/)
pnpm build:electron   # Package with Electron Builder

# Distribution (includes build)
pnpm dist             # Build for current platform
pnpm dist:mac         # macOS (.dmg)
pnpm dist:win         # Windows (.exe, portable)
pnpm dist:linux       # Linux (.AppImage, .deb, .rpm)

# Testing
pnpm test:platforms   # Cross-platform tests (includes Docker)
pnpm test:paths       # Path handling test
```

### Hot Reload Behavior

- **Renderer Process (Frontend):** Full HMR via Vite
  - Changes to `src/renderer/**` → Instant reload
  - No app restart needed

- **Main Process (Backend):** Manual restart required
  - Changes to `electron/**` → Must restart `pnpm dev`
  - Or use `nodemon`/`electron-reloader` (not configured by default)

### Adding New IPC Handlers

1. **Define handler in main process** (`electron/main.js` or relevant module):
```javascript
// electron/newFeature.js
async function handleNewFeature(arg) {
  // Implementation
  return result;
}

module.exports = { handleNewFeature };

// electron/main.js
const { handleNewFeature } = require('./newFeature');
ipcMain.handle('new-feature', async (event, arg) => {
  return await handleNewFeature(arg);
});
```

2. **Expose in preload** (`electron/preload.js`):
```javascript
contextBridge.exposeInMainWorld('electron', {
  // ... existing APIs
  newFeature: (arg) => ipcRenderer.invoke('new-feature', arg)
});
```

3. **Use in renderer** (`src/renderer/App.jsx` or component):
```javascript
const handleClick = async () => {
  const result = await window.electron.newFeature(arg);
  console.log(result);
};
```

### Adding New React Components

1. **Create component file** (`src/renderer/components/NewComponent.jsx`):
```jsx
import React from 'react';

export default function NewComponent({ prop1, prop2 }) {
  return (
    <div>
      {/* Component JSX */}
    </div>
  );
}
```

2. **Import and use:**
```jsx
import NewComponent from '@/components/NewComponent';

function App() {
  return <NewComponent prop1="value" prop2={42} />;
}
```

**Note:** The `@` alias resolves to `src/renderer/` (configured in `vite.config.cjs`).

### Adding New MCP Server

1. **Open Settings page** in the app
2. **Navigate to MCP Servers section**
3. **Click "Add Server"**
4. **Configure:**
   - **Server ID:** Unique identifier (e.g., `my-server`)
   - **Transport Type:** `stdio`, `sse`, or `streamableHttp`
   - **Command/URL:** Depends on transport type
     - STDIO: `node`, `python`, `docker`, `npx`, `uvx`, `deno`
     - SSE/HTTP: Full URL
   - **Args:** Array of arguments (STDIO only)
   - **Env:** Environment variables (STDIO only)
   - **Headers:** Custom headers (SSE/HTTP only)
   - **Auth:** OAuth configuration (if required)

5. **Save and Connect**

**Storage:** Saved to `userData/settings.json` under `mcpServers` key.

## Code Conventions

### File Naming
- **React components:** PascalCase (e.g., `ChatInput.jsx`, `ToolsPanel.jsx`)
- **Utilities/helpers:** camelCase (e.g., `utils.js`, `messageUtils.js`)
- **Managers/handlers:** camelCase (e.g., `mcpManager.js`, `chatHandler.js`)
- **Config files:** kebab-case (e.g., `vite.config.cjs`, `tailwind.config.cjs`)

### Code Style

#### JavaScript/JSX
```javascript
// Use ESLint flat config (eslint.config.js)
// Rules:
// - Unused vars: warn (allow _prefixed)
// - Prop types: off (using TypeScript patterns, not enforced)
// - JSX runtime: automatic (no need to import React)

// Formatting preferences
// - 2-space indentation
// - Single quotes for strings
// - Semicolons required
// - Trailing commas in multiline
```

#### React Patterns
```jsx
// Prefer functional components with hooks
function MyComponent({ prop1, prop2 }) {
  const [state, setState] = useState(initialValue);

  useEffect(() => {
    // Side effects
  }, [dependencies]);

  return <div>{/* JSX */}</div>;
}

// Use context for global state
const { messages, setMessages } = useContext(ChatContext);

// Use refs for values that don't trigger re-renders
const currentChatIdRef = useRef(null);
```

#### Styling
```jsx
// Use Tailwind utility classes
<div className="flex items-center gap-2 p-4 bg-gray-100 rounded-lg">

// Use cn() helper for conditional classes
import { cn } from '@/lib/utils';

<div className={cn(
  'base-classes',
  isActive && 'active-classes',
  variant === 'primary' && 'primary-classes'
)}>
```

#### Error Handling
```javascript
// Main process: Log errors, return error objects
try {
  const result = await riskyOperation();
  return { success: true, data: result };
} catch (error) {
  console.error('Operation failed:', error);
  return { success: false, error: error.message };
}

// Renderer: Show user-friendly errors
try {
  await window.electron.someOperation();
} catch (error) {
  console.error(error);
  alert(`Error: ${error.message}`);
  // Or use toast/notification system
}
```

### Git Workflow

```bash
# 1. Create feature branch
git checkout -b feature/my-feature

# 2. Make changes, commit frequently
git add .
git commit -m "feat: add new feature"

# 3. Push to remote
git push origin feature/my-feature

# Commit message format (conventional commits)
# feat: New feature
# fix: Bug fix
# docs: Documentation
# style: Formatting
# refactor: Code restructuring
# test: Tests
# chore: Maintenance
```

## Common Tasks

### Add New UI Component

```bash
# Using shadcn-style pattern

# 1. Create component file
# src/renderer/components/ui/my-component.jsx

import { cn } from '@/lib/utils';

export function MyComponent({ className, ...props }) {
  return (
    <div className={cn('base-styles', className)} {...props}>
      {props.children}
    </div>
  );
}
```

### Add New Settings Field

```javascript
// 1. Update Settings.jsx UI
<div>
  <label>New Setting</label>
  <input
    value={settings.newSetting || ''}
    onChange={(e) => handleSettingChange('newSetting', e.target.value)}
  />
</div>

// 2. Default value (optional)
// In electron/settingsManager.js
const DEFAULT_SETTINGS = {
  // ... existing
  newSetting: 'default-value'
};

// 3. Use in code
const settings = await window.electron.getSettings();
console.log(settings.newSetting);
```

### Modify UI Styling

```javascript
// Option 1: Modify global styles
// Edit: src/renderer/index.css

// Option 2: Modify Tailwind configuration
// Edit: tailwind.config.cjs

// Option 3: Update component-specific styles
// Edit the component file and update className attributes
```

## Performance Considerations

### Message Pruning
- Automatically prunes messages to 50% of context window
- Uses rough estimation: chars * 0.3 = tokens
- Vision images counted as ~800 tokens
- Keeps recent message groups intact

### MCP Health Checks
- Every 60 seconds
- Only calls `listTools()` (lightweight)
- Auto-disconnects unhealthy servers
- Prevents zombie connections

### Streaming Optimizations
- Progressive UI updates (not waiting for full response)
- Auto-scroll only when user at bottom
- Uses `requestAnimationFrame` for smooth scrolling
- Cleanup event listeners on completion

### Token Caching
- Model list cached for 5 minutes
- Reduces API calls
- Stale cache used on fetch failure

### Local Storage
- Chat history stored as individual files (not one large file)
- Metadata-only loading for chat list
- Full messages loaded on-demand
- Settings debounced on save

## Security Considerations

### Context Isolation
- **Enabled** via `contextIsolation: true` in BrowserWindow
- Renderer cannot access Node.js APIs directly
- All communication via `contextBridge` in preload.js

### Node Integration
- **Disabled** via `nodeIntegration: false`
- Prevents arbitrary code execution in renderer

### Remote Content
- Only loads from `localhost:5173` (dev) or `file://` (prod)
- No external URLs loaded in main window

### API Key Storage
- Stored in `userData/settings.json` (OS-protected directory)
- Not exposed to renderer (sent via IPC only)
- Never logged or sent to untrusted endpoints

### OAuth Tokens
- Refresh tokens stored in `electron-json-storage`
- Access tokens refreshed automatically
- State verification in OAuth flows (prevents CSRF)

### Tool Execution
- User approval required (default)
- Per-tool approval persistence (localStorage in renderer)
- Sandboxed MCP servers (separate processes)

## Debugging

### Main Process Debugging

```bash
# 1. Add debugger statements or console.log
console.log('Main process:', data);

# 2. Open DevTools for main process
# In electron/main.js, add:
require('electron-debug')({ showDevTools: true });

# Or use Chrome DevTools
electron --inspect=5858 .
# Then open chrome://inspect in Chrome
```

### Renderer Process Debugging

```bash
# DevTools automatically open in development mode
# Or manually in app: View → Toggle Developer Tools

# React DevTools
# Install extension: https://react.dev/learn/react-developer-tools
```

### API Request Logging

```javascript
// Enable in Settings
settings.enableApiLogging = true;

// Logs saved to: /tmp/groq-api-request-YYYYMMDD-HHMMSS-ID.json
// Contains: model, messages, tools, response, timestamps
```

## Key Files Reference

### Configuration Files

| File | Purpose |
|------|---------|
| `package.json` | Dependencies, scripts, Electron Builder config |
| `vite.config.cjs` | Vite bundler configuration |
| `electron-builder.yml` | Electron Builder packaging settings |
| `tailwind.config.cjs` | Tailwind CSS configuration |
| `postcss.config.cjs` | PostCSS configuration |
| `eslint.config.js` | ESLint rules (flat config) |
| `pnpm-workspace.yaml` | pnpm workspace settings |

### Entry Points

| File | Purpose |
|------|---------|
| `electron/main.js` | Electron main process entry |
| `electron/preload.js` | Context bridge (security) |
| `src/renderer/main.jsx` | React app entry |
| `src/renderer/App.jsx` | Main React component |
| `index.html` | HTML entry point |

### Core Backend Modules (electron/)

| File | Purpose |
|------|---------|
| `chatHandler.js` | Groq API communication |
| `mcpManager.js` | MCP server lifecycle |
| `toolHandler.js` | Tool execution |
| `authManager.js` | MCP OAuth flows |
| `googleOAuthManager.js` | Google OAuth refresh |
| `chatHistoryManager.js` | Chat persistence |
| `settingsManager.js` | Settings persistence |
| `commandResolver.js` | Platform-aware commands |
| `messageUtils.js` | Message pruning |

### Core Frontend Components (src/renderer/)

| File | Purpose |
|------|---------|
| `App.jsx` | Main app orchestration |
| `context/ChatContext.jsx` | Global chat state |
| `pages/Settings.jsx` | Settings UI |
| `components/ChatInput.jsx` | Message input |
| `components/MessageList.jsx` | Message display |
| `components/Message.jsx` | Individual message |
| `components/ToolsPanel.jsx` | MCP tools UI |
| `components/ToolApprovalModal.jsx` | Tool permissions |

## Resources

### Official Documentation
- **Electron:** https://www.electronjs.org/docs
- **React:** https://react.dev/
- **Vite:** https://vitejs.dev/
- **Tailwind CSS:** https://tailwindcss.com/docs
- **Groq API:** https://console.groq.com/docs
- **MCP Specification:** https://modelcontextprotocol.io/

### Key Dependencies
- **groq-sdk:** https://github.com/groq/groq-typescript
- **@modelcontextprotocol/sdk:** https://github.com/modelcontextprotocol/sdk
- **electron-builder:** https://www.electron.build/
- **React Router:** https://reactrouter.com/

---

## Quick Reference: Common Patterns

### IPC Call Pattern
```javascript
// Renderer (Frontend)
const result = await window.electron.someFunction(arg);

// Main (Backend)
ipcMain.handle('some-function', async (event, arg) => { return result; });

// Preload (Bridge)
contextBridge.exposeInMainWorld('electron', {
  someFunction: (arg) => ipcRenderer.invoke('some-function', arg)
});
```

### Add Message to Chat
```javascript
const { messages, setMessagesWithSave } = useContext(ChatContext);

const newMessage = {
  role: 'user',
  content: 'Hello!'
};

setMessagesWithSave([...messages, newMessage]);
// Auto-saves to disk
```

### Execute Tool
```javascript
const toolCall = {
  id: 'call_123',
  type: 'function',
  function: { name: 'tool_name', arguments: '{"param": "value"}' }
};

const result = await window.electron.executeToolCall(toolCall);
// Returns: { role: 'tool', tool_call_id: 'call_123', content: '...' }
```

### Get/Update Settings
```javascript
// Get
const settings = await window.electron.getSettings();

// Update
settings.someKey = 'newValue';
await window.electron.saveSettings(settings);
```

### Connect MCP Server
```javascript
await window.electron.connectMCPServer(serverId);

// Check status
const status = await window.electron.getMCPStatus();
console.log(status[serverId]); // { status: 'connected', tools: [...] }
```

---

## Agent-Specific Notes

### For Code Generation
- Always read files before modifying them
- Follow existing code patterns and conventions
- Use the same formatting style (2 spaces, single quotes, semicolons)
- Test changes in development mode (`pnpm dev`)

### For Code Review
- Check for security issues (context isolation, API key exposure)
- Verify error handling is present
- Ensure IPC communication follows the preload pattern
- Verify React components use hooks properly

### For Documentation
- Update this file when making architectural changes
- Keep code examples up to date
- Document new IPC handlers in the relevant sections
- Update version numbers and last updated date

### For Troubleshooting
- Check console logs in both main and renderer processes
- Enable API logging for debugging API issues
- Use DevTools for frontend debugging
- Check MCP server connection status in Tools Panel

---

**Last Updated:** 2025-12-10
**Repository:** groq-desktop-beta-windows
**Target Audience:** All LLM agents (Claude, GPT, Llama, Gemini, etc.)
