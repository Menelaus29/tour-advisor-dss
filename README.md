# Tour Advisor DSS

A decision support system (DSS) that helps travelers choose a tour package.

## Tech stack

| Part | Technology |
|---|---|
| Frontend | Next.js (App Router), TypeScript, Tailwind CSS |
| Backend | NestJS (ESM), TypeScript |
| ORM | Prisma 7 with the `pg` driver adapter |
| Database | PostgreSQL: local via Docker for development, Supabase for the shared cloud/demo database |

The frontend (port 3000) calls the backend (port 3001) over REST.

## Repository layout

```
frontend/             Next.js app
backend/              NestJS app
docs/                 
  prisma/             schema.prisma and migrations
  src/prisma/         PrismaService and PrismaModule
.editorconfig
.gitignore
.nvmrc
README.md
docker-compose.yml    local PostgreSQL for development
```

## Prerequisites

- Git
- Node.js (current LTS - see `.nvmrc`) and npm
- Docker Desktop (for the local database)

## Getting started

**1. Start the local database** (from the repo root):

```bash
docker compose up -d
```

**2. Backend** (in `backend/`):

```bash
cd backend
cp .env.example .env        # Windows: copy .env.example .env
npm install
```

Open `backend/.env` and set the local connection string:

```dotenv
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/tour_dss"
```

Then apply the migrations, generate the Prisma client, and start the server:

```bash
npx prisma generate
npx prisma migrate deploy
npm run start:dev
```

The backend runs at http://localhost:3001 and should answer "Hello World!".

**3. Frontend** (second terminal, in `frontend/`):

```bash
cd frontend
cp .env.example .env.local  # Windows: copy .env.example .env.local
npm install
npm run dev
```

The frontend runs at http://localhost:3000.

## Database workflow

- **Develop against the local Docker database**, not the Supabase one.
- **Changing the schema:** edit `backend/prisma/schema.prisma`, run `npx prisma migrate dev --name <short_description>`, then commit the schema change and the new folder in `backend/prisma/migrations/`.
- **After pulling someone else's migrations:** run `npx prisma migrate deploy`, then `npx prisma generate`.
- **The generated Prisma client** (`backend/src/generated/prisma`) is not committed. Run `npx prisma generate` after `npm install` and whenever the schema changes.
- **The Supabase database** is for deployment and demos only. Its credentials are held by the backend owner, so you don't need them. Migrations are applied there with `npx prisma migrate deploy` only. Never run `migrate dev` or `migrate reset` against it.

Useful Docker commands:

```bash
docker compose down        # stop the database (data is kept)
docker compose down -v     # stop and delete all local database data
```

## Troubleshooting

- **Port already in use** (3000, 3001, or 5432): if you already run Postgres locally on 5432, create a `.env` file in the repo root containing `DB_PORT=5433`, run `docker compose up -d` again, and use `localhost:5433` in `DATABASE_URL`.
- **Cannot find `generated/prisma/client.js`:** run `npx prisma generate` in `backend/`.
- **npm warns that Prisma install scripts are blocked, or Prisma reports a missing engine:** run `npm rebuild prisma @prisma/engines` in `backend/`.
