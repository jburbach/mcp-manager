# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP Manager is a web-based Docker container management tool specifically designed for managing Model Context Protocol (MCP) servers. The project is organized into 4 development phases:

- **Phase 1** (Current): Core Foundation - Docker container with basic web UI and server management
- **Phase 2**: Server Management - Traditional process management and health monitoring  
- **Phase 3**: User Interface - Enhanced UI with real-time updates
- **Phase 4**: Polish and Release - Security, documentation, and deployment

## Current Development Status

**Active Phase**: Phase 1 - Core Foundation  
**Current Sprint**: Week 1 - Project Setup & Docker Container  
**Priority**: P0 - Docker integration and basic UI

The codebase is currently in early development with only planning documents completed. No source code has been implemented yet.

## Architecture Overview

The system follows a containerized web application architecture:

```
┌─────────────────────────────────────┐
│        MCP Manager Container        │
├─────────────────┬───────────────────┤
│   Frontend      │   Backend API     │
│   (React)       │   (Node.js)       │
│   Port 3000     │   Port 3001       │
└─────────────────┴───────────────────┘
          │
    Docker Socket Mount
          │
┌─────────────────────────────────────┐
│      Host Docker Engine             │
│  ┌─────────┐ ┌─────────┐           │
│  │MCP-1    │ │MCP-2    │           │
│  │Container│ │Container│           │
│  └─────────┘ └─────────┘           │
└─────────────────────────────────────┘
```

## Technology Stack

### Backend
- **Runtime**: Node.js 18+
- **Framework**: Express.js
- **Database**: SQLite (single file)
- **Docker Integration**: dockerode library
- **WebSocket**: ws library for real-time updates

### Frontend
- **Framework**: React 18 with TypeScript
- **Styling**: Tailwind CSS
- **HTTP Client**: axios
- **WebSocket**: native WebSocket API

### Infrastructure
- **Container**: Alpine Linux base
- **Process Manager**: Node.js native
- **Storage**: Volume mounts for persistence

## Core Database Schema

```sql
CREATE TABLE servers (
  id TEXT PRIMARY KEY DEFAULT (hex(randomblob(16))),
  name TEXT NOT NULL UNIQUE,
  type TEXT NOT NULL DEFAULT 'container',
  image TEXT, -- Docker image name
  status TEXT NOT NULL DEFAULT 'stopped', -- 'running', 'stopped', 'error'
  port INTEGER, -- Exposed port
  container_id TEXT, -- Docker container ID
  config TEXT NOT NULL DEFAULT '{}', -- JSON configuration
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

## Key API Endpoints

```
# Health check
GET /health

# Server management
GET /api/servers
POST /api/servers
GET /api/servers/:id
PUT /api/servers/:id
DELETE /api/servers/:id
POST /api/servers/:id/start
POST /api/servers/:id/stop
POST /api/servers/:id/restart

# System info
GET /api/system/info
```

## Core TypeScript Interfaces

```typescript
interface MCPServer {
  id: string;
  name: string;
  type: 'container';
  image: string;
  status: 'running' | 'stopped' | 'error';
  port?: number;
  containerId?: string;
  config: ServerConfig;
  createdAt: string;
  updatedAt: string;
}

interface ServerConfig {
  environment?: Record<string, string>;
  volumes?: string[];
  ports?: Record<string, string>;
}
```

## Component Structure

```
src/
├── components/
│   ├── Layout/
│   │   ├── Header.tsx
│   │   ├── Sidebar.tsx
│   │   └── Layout.tsx
│   ├── Common/
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Modal.tsx
│   │   ├── LoadingSpinner.tsx
│   │   └── StatusIndicator.tsx
│   └── Dashboard/
│       ├── ServerCard.tsx
│       ├── ServerList.tsx
│       └── Dashboard.tsx
├── pages/
│   ├── Dashboard.tsx
│   ├── Servers.tsx
│   └── Settings.tsx
├── hooks/
│   ├── useApi.ts
│   ├── useServers.ts
│   └── useWebSocket.ts
├── services/
│   ├── api.ts
│   └── websocket.ts
├── types/
│   ├── server.ts
│   └── api.ts
└── App.tsx
```

## Docker Integration Requirements

All MCP containers must be labeled with:
- `mcp.managed=true` - Identifies containers managed by MCP Manager
- `mcp.server.id=<server-id>` - Links container to database record

Container creation follows this pattern:
- RestartPolicy: 'unless-stopped'
- Port bindings for exposed services
- Environment variable injection
- Volume mounts as needed

## Development Commands

Since the project is in early development, standard commands are not yet established. When implementing:

- Use `npm` as the package manager
- Implement standard scripts: `start`, `dev`, `build`, `test`, `lint`
- Follow dockerode documentation for Docker API integration
- Use sqlite3 for database operations

## Key Development Principles

1. **Container-First Design**: All MCP servers run as Docker containers with proper labeling
2. **Real-time Updates**: UI reflects container state changes via WebSocket
3. **Validation**: Server configurations must be validated before container creation
4. **Error Handling**: Docker operations can fail - implement proper error recovery
5. **Status Sync**: Database state must stay synchronized with actual container state

## Phase 1 Requirements Summary

The current phase focuses on:
- Docker container that runs MCP Manager
- Basic React UI for server management
- SQLite database for persistence
- Docker API integration for container lifecycle
- Server discovery and registration
- Start/stop/restart operations

Phase 1 excludes authentication, advanced monitoring, and traditional process management - these are planned for later phases.