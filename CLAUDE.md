# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run setup      # Install dependencies + generate Prisma client + run migrations
npm run dev        # Start dev server with Turbopack (http://localhost:3000)
npm run build      # Production build
npm run lint       # ESLint
npm test           # Run all tests with Vitest
npm test -- --run  # Run tests once (no watch)
npm test -- path/to/file.test.ts  # Run specific test file
npm run db:reset   # Reset database (destructive)
```

## Architecture

UIGen is an AI-powered React component generator with live preview. Users describe components in chat, Claude generates code using tools, and the preview updates in real-time.

### Core Data Flow

1. **Chat API** (`src/app/api/chat/route.ts`): Streams AI responses using Vercel AI SDK with two custom tools:
   - `str_replace_editor`: Create/edit files (view, create, str_replace, insert commands)
   - `file_manager`: Rename/delete files

2. **Virtual File System** (`src/lib/file-system.ts`): In-memory file system that never writes to disk. Serializes to JSON for persistence.

3. **Context Providers** (`src/lib/contexts/`):
   - `FileSystemProvider`: Manages VirtualFileSystem instance and handles tool calls
   - `ChatProvider`: Wraps Vercel AI SDK's useChat, sends serialized files with each request

4. **Live Preview** (`src/components/preview/PreviewFrame.tsx` + `src/lib/transform/jsx-transformer.ts`):
   - Transforms JSX/TSX using Babel standalone in browser
   - Creates import map with blob URLs for each file
   - Third-party imports resolve to esm.sh
   - Renders in sandboxed iframe

### Database

SQLite via Prisma. Schema in `prisma/schema.prisma`:

- `User`: Email/password auth
- `Project`: Stores serialized messages and file system data as JSON strings

### Key Patterns

- Path alias: `@/*` maps to `./src/*`
- UI components use shadcn/ui (new-york style) in `src/components/ui/`
- Server actions in `src/actions/` for project CRUD
- Auth uses jose for JWT tokens (`src/lib/auth.ts`)
- Tests colocated with components in `__tests__/` directories
- Mock provider: Without `ANTHROPIC_API_KEY`, the app returns static responses instead of calling Claude
- Use comments sparingly. Only comment complex code.
