# Nexa - AI App Builder

Describe what you want to build. AI writes the code, picks the packages, and renders a live preview -- all inside your browser.

Built with Next.js 16, Google Gemini, Sandpack, and Clerk.

---

## Overview

Nexa is an AI-powered no-code/low-code application builder. Users describe their app idea in plain English, and the platform uses Google Gemini to generate production-ready React + Tailwind CSS code. The result renders instantly in a live Sandpack preview, and users can iterate by chatting further with the AI.

The platform includes authentication, a credit-based billing system, project history, image-aware prompts, AI error recovery, and a Pro-tier agentic improvement feature powered by the Cline SDK.

---

## Features

- **AI App Generation** -- Describe your app in natural language. Gemini generates complete React components with Tailwind CSS styling.
- **Live Preview via Sandpack** -- Every generation renders instantly in an interactive browser preview powered by CodeSandbox Sandpack.
- **Project Saving** -- Every workspace is persisted to PostgreSQL. Return anytime to continue where you left off.
- **Image-Aware Prompts** -- Attach screenshots or mockups. Gemini reads the image and generates code that matches your design.
- **Smart Package Validation** -- AI-selected npm dependencies are validated against the npm registry; hallucinated packages are silently filtered.
- **AI Error Recovery** -- When a preview throws an error, a banner appears. One click sends the error back to AI for an auto-fix.
- **Improve with Agent (Pro)** -- A Cline-based agentic workflow that analyzes your codebase and applies targeted improvements via a tool-calling loop.
- **Export to ZIP** -- Download your generated app as a complete project with package.json, src files, and Tailwind CDN setup.
- **Credit-Based Billing** -- Free tier with 10 generations/month, paid plans for heavier usage.
- **Authentication** -- Secure sign-in/sign-up via Clerk.
- **Project Dashboard** -- Browse, open, and delete past projects from a central projects page.

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Framework** | Next.js 16 (App Router) |
| **Language** | TypeScript |
| **Styling** | Tailwind CSS v4, `tw-animate-css`, `framer-motion`, `motion` |
| **UI Components** | shadcn/ui (based on `@base-ui/react`), Lucide icons |
| **Authentication** | Clerk (`@clerk/nextjs`) |
| **AI Provider** | Google Gemini (`gemini-2.5-flash` via `@google/genai`) |
| **Agentic SDK** | Cline SDK (`@cline/sdk`) for Pro-tier improvement agent |
| **Database** | PostgreSQL via Supabase, Prisma ORM |
| **Live Preview** | CodeSandbox Sandpack (`@codesandbox/sandpack-react`) |
| **Image Storage** | Supabase Storage |
| **Rate Limiting / Security** | Arcjet (`@arcjet/next`) |
| **Billing** | Clerk Checkout (Stripe-backed) |
| **Export** | JSZip |
| **Deployment** | Vercel (Fluid compute) |

---

## Project Structure

```
nexa-agentic-app-builder/
├── actions/                   # Server Actions (workspace, projects)
│   ├── projects.ts
│   └── workspace.ts
├── app/
│   ├── (auth)/                # Auth layout + pages
│   │   ├── sign-in/
│   │   └── sign-up/
│   ├── (main)/                # Authenticated layout + pages
│   │   ├── projects/
│   │   └── workspace/
│   ├── api/
│   │   ├── gen-ai-code/       # POST - Gemini code generation (SSE stream)
│   │   └── improve/           # POST - Cline agent improvement (SSE stream)
│   ├── globals.css
│   ├── layout.tsx             # Root layout (Clerk + ThemeProvider)
│   └── page.tsx               # Landing page
├── components/
│   ├── animate-ui/            # Animated background components
│   ├── ui/                    # shadcn/ui primitives
│   ├── ChatPanel.tsx          # Chat interface with image upload
│   ├── CodePanel.tsx          # Sandpack editor + preview
│   ├── DeleteProjectModal.tsx
│   ├── Header.tsx             # Top navigation
│   ├── PricingModal.tsx       # Plan selection dialog
│   ├── ProjectCard.tsx        # Project grid card
│   ├── WorkspaceClient.tsx    # Main workspace orchestrator
│   ├── reusables.tsx          # Shared styled components
│   └── theme-provider.tsx
├── lib/
│   ├── arcjet.ts              # Arcjet rate limiting + prompt injection
│   ├── checkUser.ts           # Clerk user sync with DB
│   ├── constants.ts           # Plan definitions, credit costs
│   ├── data.ts                # Landing page data (features, steps, suggestions)
│   ├── prisma.ts              # Prisma client singleton
│   ├── utils.ts               # cn() utility
│   └── generated/prisma/      # Auto-generated Prisma client
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── types/
│   ├── index.ts
│   ├── plans.ts
│   ├── project.ts
│   └── workspace.ts
├── proxy.ts                   # Clerk middleware + route protection
├── next.config.ts
├── vercel.json
└── tsconfig.json
```

---

## Installation & Setup

### Prerequisites

- Node.js >= 18
- PostgreSQL database (Supabase recommended)
- Clerk account
- Google Gemini API key
- Supabase account (for image storage)
- Arcjet account (optional, for rate limiting)

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/your-org/nexa-agentic-app-builder.git
cd nexa-agentic-app-builder

# 2. Install dependencies
npm install

# 3. Copy environment variables
cp .env.example .env.local

# 4. Fill in your .env.local (see Environment Variables section)

# 5. Run database migrations
npx prisma migrate deploy

# 6. Start the dev server
npm run dev
```

Visit `http://localhost:3000` to see the application.

---

## Environment Variables

```env
# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up

# PostgreSQL (Supabase)
DATABASE_URL="postgresql://..."
DIRECT_URL="postgresql://..."

# AI
GEMINI_API_KEY=

# Supabase (image uploads)
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=

# Rate Limiting / Security (Arcjet)
ARCJET_KEY=

# Optional: Cline SDK uses GEMINI_API_KEY by default
```

---

## How It Works

### 1. Prompt Input

The user describes their app idea on the landing page or in the workspace chat panel. Example suggestions are provided for quick start. Image attachments are uploaded to Supabase Storage and passed to the AI as context.

### 2. AI Code Generation

The prompt is sent to `/api/gen-ai-code` which:

1. Validates authentication and checks user credits.
2. Builds a conversation history with system instructions.
3. Streams the prompt to Gemini (`gemini-2.5-flash`) with `thinkingConfig` enabled.
4. Gemini returns a structured JSON response containing files, dependencies, and an assistant message.
5. Dependencies are validated against the npm registry.
6. The workspace is upserted in PostgreSQL, and a credit is deducted in a single transaction.
7. A Server-Sent Events (SSE) stream delivers status updates, then the final file data.

### 3. Live Preview

The returned files are loaded into a Sandpack React template. The user can toggle between **Preview** (live rendered app) and **Code** (file explorer + read-only editor). Any runtime errors appear in a dismissible banner with a "Fix with AI" button.

### 4. Improvement Loop (Pro)

Pro users can use the **Improve with Agent** button, which triggers `/api/improve`. This endpoint:

1. Spawns a Cline agent with access to an `update_file` tool and a `done_improving` tool.
2. The agent analyzes the current codebase and user request, then calls `update_file` for each file it wants to change.
3. Each file patch is streamed to the client in real time via SSE.
4. When done, the updated files are saved and the preview refreshes.

### 5. Save, Export, and Manage

- Projects are auto-saved after each generation.
- The **Projects** page lists all workspaces with message counts and timestamps.
- The **Download** button exports the app as a ZIP archive containing a full project with `package.json`, `src/` files, and Tailwind CDN setup.

---

## Plans & Billing

Nexa uses a credit-based metering system with billing managed through Clerk Checkout (powered by Stripe).

| Plan | Price | Credits/Month | Key Features |
|---|---|---|---|
| **Free** | $0 | 10 | Live preview, export to ZIP |
| **Starter** | $9/mo | 50 | Image uploads, live preview, export |
| **Pro** | $29/mo | 150 | Priority AI, Improve with Agent, image uploads, Forge Pro Agent |

- Each generation consumes **1 credit**.
- When out of credits, an upgrade prompt is shown.
- Users can upgrade or downgrade at any time. Credits are topped up on upgrade (credit delta is calculated and added to remaining credits).

---

## API Routes

### `POST /api/gen-ai-code`

Generates a new app or continues an existing conversation.

**Request body:**
```json
{
  "workspaceId": "string | null",
  "userId": "string",
  "messages": [{ "role": "user|assistant", "content": "string" }],
  "fileData": { "files": {}, "dependencies": {} } | null
}
```

**Response:** SSE stream with events:
- `status` -- progress updates (e.g., "Thinking...", "Validating packages...")
- `done` -- final result with workspaceId, fileData, creditsRemaining
- `error` -- error message

### `POST /api/improve`

Improves existing code using a Cline agent (Pro only).

**Request body:**
```json
{
  "userId": "string",
  "workspaceId": "string",
  "userRequest": "string",
  "fileData": { "files": {}, "dependencies": {} }
}
```

**Response:** SSE stream with events:
- `thinking` -- agent reasoning text
- `file_patch` -- live file updates applied in real time
- `done` -- final result with updated fileData and summary
- `error` -- error message

### Server Actions

- `getUserProjects()` -- Returns all workspaces for the authenticated user.
- `deleteProject(workspaceId)` -- Deletes a workspace (owner only).
- `getWorkspaceUser()` -- Returns current user's id, credits, and plan.
- `getWorkspaceById(id, userId)` -- Returns workspace data (owner only).

---

## Deployment

### Vercel (Recommended)

1. Push the repository to GitHub.
2. Import the project in Vercel.
3. Set all environment variables listed above.
4. Set the `maxDuration` for the API routes -- the `vercel.json` enables **Fluid** compute which supports up to 300s function duration.
5. Deploy.

### Database

The app uses a PostgreSQL database managed by Prisma. Migrations are located in `prisma/migrations/`. Run `npx prisma migrate deploy` before starting production.

### Environment Notes

- The API routes use `export const runtime = "nodejs"` (required for the PostgreSQL adapter).
- Image uploads use Supabase Storage with a public bucket named `workspace-bucket`.
- The `@cline/sdk` packages are listed as `serverExternalPackages` in `next.config.ts`.

---

## Future Improvements

1. **Custom domain generation** -- Allow users to deploy generated apps to a unique subdomain or custom domain.
2. **Team workspaces** -- Multi-user collaboration on the same project with role-based permissions.
3. **AI chat memory** -- Longer conversation history with persistent context across sessions.
4. **Component library** -- Curated set of pre-built, customizable UI blocks that users can compose via prompts.
5. **One-click deploy** -- Direct deployment to Vercel, Netlify, or Cloudflare via OAuth integration.
6. **API generation** -- Generate full backend APIs (REST/GraphQL) alongside the frontend.
7. **Version history** -- Snapshot-based versioning to roll back to any previous generation.
8. **Prompt marketplace** -- Community-shared prompts and templates for common app types.
9. **Dark mode enhancements** -- Toggle theme preview within generated apps.
10. **Mobile responsive editor** -- Optimized workspace for tablets and mobile devices.

---

## Contributing

Contributions are welcome!

1. Fork the repository.
2. Create a feature branch: `git checkout -b feat/my-feature`.
3. Commit your changes: `git commit -m 'Add my feature'`.
4. Push to the branch: `git push origin feat/my-feature`.
5. Open a Pull Request.

Please ensure your code follows the existing style conventions and passes linting (`npm run lint`).

---

## License

MIT License

Copyright (c) 2026 Asif Ahmed Sahil

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
