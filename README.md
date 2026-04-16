# Next.js + Express Monorepo

Full-stack application with authentication and user management.

## Prerequisites

- Node.js 20+ and pnpm (for local development)
- Docker and Docker Compose (for containerized usage)

## Getting Started

### Option A - Local development

#### 1. External dependencies

You need `maildev` running to capture emails (e.g., password resets):

```bash
docker run -d -p 1080:1080 -p 1025:1025 --name maildev maildev/maildev
```

- **Web UI**: http://localhost:1080
- **SMTP**: localhost:1025

#### 2. Installation & setup

```bash
# Install dependencies
pnpm install

# Copy environment variables
cp .env.example .env

# Seed the database with an admin user (and default permissions)
pnpm seed

# Start development servers
pnpm dev
```

The frontend runs on `http://localhost:${FRONTEND_PORT:-3000}` and the backend on `http://localhost:${PORT:-3001}`.

### Option B - Docker Compose

#### Production mode (default)

```bash
cp .env.example .env
# Edit .env - set SESSION_SECRET and any other required values

docker compose up --build
```

On first run, seed the database from inside the running container:

```bash
docker compose exec backend pnpm seed
```

> For production deployments, set `WEBSITE_URL` and `API_URL` in `.env` to your actual HTTPS URLs before building. The frontend image must be rebuilt when `API_URL` changes, as Next.js bakes it in at build time.

#### Dev mode (hot reload)

Add this line to your `.env` to enable source mounting and hot reload:

```
COMPOSE_FILE=docker-compose.yml:docker-compose.dev.yml
```

Then run as normal:

```bash
docker compose up --build
```

Changes to files under `packages/backend/src` and `packages/frontend/src` are picked up automatically without rebuilding the image.

## Scripts

| Command | Description |
| --- | --- |
| `pnpm dev` | Start both frontend and backend in dev mode |
| `pnpm seed` | Create initial admin user and permissions |
| `pnpm build` | Build the project |

## Project Structure

```
packages/
  backend/           Express API server
    src/
      config/        Database configuration
      middleware/    Auth middleware (isAuthenticated, hasPermission)
      models/        Sequelize models (User, Permission, UserPermission)
      passport/      Passport strategies (local, Google OAuth)
      routes/        API routes (auth, users, feature-flags, blood-pressure)
      scripts/       CLI scripts (seed)
  frontend/          Next.js application
    src/
      app/           Pages (home, login, profile, users)
      components/    UI components and navbar
      context/       React context providers (auth)
      hooks/         Custom hooks (useRequireAuth)
      lib/           Utilities, API client, types
```

## Database & Sequelize

The backend uses **Sequelize** with **SQLite**. The database file is stored at `database/database.sqlite` in the project root (outside the backend package so it can be easily volume-mounted in Docker). The path can be overridden with the `DB_PATH` environment variable.

### Initialization

When you run the server (`pnpm dev` or `tsx src/index.ts`), the application automatically syncs the database models with the database file.
This `sequelize.sync()` call will create tables if they do not exist.

### Seeding

To populate the database with initial data (Admin user, default permissions, feature flags):

```bash
pnpm seed
```

This runs `packages/backend/src/scripts/seed.ts`.

### Migrations

For schema changes (adding columns, new tables) that you want to version control, use `sequelize-cli`.

> [!IMPORTANT]
> You must run these commands from the `packages/backend` directory.

#### Creating a Migration

To create a new migration file:

```bash
cd packages/backend
pnpm exec sequelize-cli migration:generate --name <migration-name>
# Example: pnpm exec sequelize-cli migration:generate --name add-age-to-users
```

This creates a file in `packages/backend/migrations`. Edit this file to define your `up` and `down` logic.

**Example 1: Adding a new column**

```javascript
// migrations/XXXXXXXXXXXXXX-add-age-to-users.js
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.addColumn('Users', 'age', {
      type: Sequelize.INTEGER,
      allowNull: true,
    });
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.removeColumn('Users', 'age');
  }
};
```

**Example 2: Creating a new table**

```javascript
// migrations/XXXXXXXXXXXXXX-create-posts.js
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('Posts', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      title: {
        type: Sequelize.STRING,
        allowNull: false
      },
      userId: {
        type: Sequelize.INTEGER,
        references: {
          model: 'Users',
          key: 'id'
        },
        onUpdate: 'CASCADE',
        onDelete: 'SET NULL'
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable('Posts');
  }
};
```

#### Running Migrations

```bash
cd packages/backend
pnpm exec sequelize-cli db:migrate
```

#### Undoing Migrations

```bash
cd packages/backend
pnpm exec sequelize-cli db:migrate:undo
```

## Tech Stack Overview

- **Frontend**: Next.js 16, React 19, Tailwind CSS v4
- **Backend**: Express, Passport.js, Sequelize, SQLite
- **Package Manager**: pnpm workspaces