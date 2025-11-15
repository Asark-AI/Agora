# Agora GH (local workspace)

This repository uses Next.js (App Router) with TypeScript and PostgreSQL (via Prisma).

Quick start (Windows PowerShell):

```powershell
# Install
npm install

# Generate Prisma client (if you added a Prisma schema)
npm run prisma:generate

# Run migrations (development)
npm run prisma:migrate

# Run Prisma Studio (optional interactive DB UI)
npm run prisma:studio

# Run in development
npm run dev

# Build
npm run build

# Start production (after build)
npm run start
```

Environment variables

- Copy `.env.local.example` to `.env.local` and set your `DATABASE_URL` to point at your Postgres instance.

Example `DATABASE_URL` (Postgres):

postgresql://username:password@localhost:5432/agora_dev

Notes

- The project uses path aliases ("@/") with `tsconfig.json` baseUrl set to `src`.
- I replaced Firebase references in the root scaffolding with a minimal Prisma + Postgres setup. Your existing server-side code will need to be updated to use Prisma or another Postgres client instead of the current Firebase code in `src/lib/firebase.ts` and any direct Firestore usage.
- A minimal Prisma schema has been added at `prisma/schema.prisma`. Run `npm run prisma:generate` and `npm run prisma:migrate` to create the DB client and apply migrations.
- If you'd like, I can help convert specific data-access utilities (for example `src/lib/firebase.ts`) into Prisma/pg equivalents.
