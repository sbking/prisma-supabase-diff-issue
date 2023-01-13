# Prisma Supabase `migrate diff` Issue

This is a reproduction of an issue where `prisma migrate diff` generates invalid SQL for the `auth` schema on a Supabase database.

## How to reproduce

### 1. Create a Supabase project and get the database password

Create a new project on [Supabase](https://app.supabase.com). Then, go to Project Settings > Database > Connection string > URI, and copy the postgres connection URL. Replace `[YOUR-PASSWORD]` with the database password you created.

### 2. Create a shadow database

Create a shadow database for `prisma migrate`:

```sh
psql postgresql://postgres:[YOUR-PASSWORD]@db.[YOUR-PROJECT-ID].supabase.co:5432 -c 'CREATE DATABASE postgres_shadow'
```

### 3. Set up the `.env` file

Copy the `.env.example` file to `.env`:

```sh
cp .env.example .env
```

Then, replace all instances of `[YOUR-PASSWORD]` and `[YOUR-PROJECT-ID]` with the correct values.

### 4. Introspect and baseline the database

Run the following to complete the steps described in the Prisma [Introspection](https://www.prisma.io/docs/getting-started/setup-prisma/add-to-existing-project/relational-databases/introspection-typescript-postgres) and [Baseline your database](https://www.prisma.io/docs/getting-started/setup-prisma/add-to-existing-project/relational-databases/baseline-your-database-typescript-postgres) docs:

```sh
# install dependencies
npm install

# introspect the database
npx prisma db pull

# create the migrations directory
mkdir -p prisma/migrations/0_init

# generate a migration file
npx prisma migrate diff --from-empty --to-schema-datamodel prisma/schema.prisma --script > prisma/migrations/0_init/migration.sql

# mark the migration as applied
npx prisma migrate resolve --applied 0_init
```

### 5. Try to run migrations

Finally, try to run:

```sh
npx prisma migrate dev
```

This should give the following error:

```
Error: P3006

Migration `0_init` failed to apply cleanly to the shadow database.
Error:
db error: ERROR: cannot use column reference in DEFAULT expression
   0: migration_core::state::DevDiagnostic
             at migration-engine/core/src/state.rs:264
```
