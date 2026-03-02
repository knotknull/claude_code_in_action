# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. Users describe components in natural language via chat, and the AI generates code that renders in real-time.

## Commands

```bash
# Setup (install deps, generate Prisma client, run migrations)
npm run setup

# Development
cd uigen && npm run dev

# Run tests
cd uigen && npm test

# Run single test file
cd uigen && npm test -- src/lib/__tests__/file-system.test.ts

# Lint
cd uigen && npm run lint

# Reset database
cd uigen && npm run db:reset
```

## Architecture

### Virtual File System
The core abstraction is `VirtualFileSystem` (`src/lib/file-system.ts`) - an in-memory file system that stores generated code. No files are written to disk. The file system:
- Manages files/directories with create, read, update, delete, rename operations
- Provides text editor commands (view, str_replace, insert) used by AI tools
- Serializes/deserializes to JSON for persistence in the database

### AI Integration
- Chat API route (`src/app/api/chat/route.ts`) uses Vercel AI SDK with Claude
- Two tools are provided to the model:
  - `str_replace_editor`: Create files, replace text, insert lines
  - `file_manager`: Rename and delete files
- System prompt in `src/lib/prompts/generation.tsx`
- Falls back to static mock responses when `ANTHROPIC_API_KEY` is not set

### Live Preview
The preview system (`src/lib/transform/jsx-transformer.ts`, `src/components/preview/PreviewFrame.tsx`):
- Transforms JSX/TSX using Babel standalone
- Creates blob URLs for each transformed file
- Generates an import map resolving local files and npm packages via esm.sh
- Renders in a sandboxed iframe with Tailwind CSS

### State Management
- `FileSystemContext` (`src/lib/contexts/file-system-context.tsx`): Manages virtual file system state, handles tool calls from AI to update files
- `ChatContext` (`src/lib/contexts/chat-context.tsx`): Manages chat messages and AI interactions

### Database
SQLite via Prisma. Schema in `prisma/schema.prisma`:
- `User`: Email/password auth
- `Project`: Stores messages (JSON) and file system data (JSON) per project

### Auth
JWT-based authentication (`src/lib/auth.ts`) using jose. Middleware protects routes.
