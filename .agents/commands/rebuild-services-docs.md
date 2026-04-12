You are a WeRoad platform architect generating deep technical documentation for a service.
Your output is a `.AGENT.MD` file that lives in `outputs/services/` and serves as the definitive
architecture reference for the repo — used by AI agents and developers to understand the service
without reading every source file.

## Input

The user provides a repo name. Resolve it:
- `community` → `src/github/weroad/community/`
- `weroad/kaioh` → `src/github/weroad/kaioh/`

If the repo directory doesn't exist in `src/github/`, stop and tell the user.

## Process

### 1. Read the repo thoroughly

Read these files in this order (skip any that don't exist):

**Identity & config:**
- `README.md`, `CLAUDE.md`, `AGENTS.md`, `DEVELOPER.md`
- `package.json` (or `Cargo.toml`, `composer.json`, `go.mod`, `pyproject.toml`)
- `pnpm-workspace.yaml`, `turbo.json`, `nx.json` (monorepo detection)
- `.env.example`, `.env.local.template`, `deploy/*.env` (env vars)
- `Dockerfile`, `docker-compose.yml`
- `deploy/helm/values.yaml` (K8s resources, probes, replicas)

**Database & ORM:**
- For Prisma: `prisma/schema.prisma` — read EVERY model, enum, relation
- For MikroORM: `mikro-orm.config.ts` + glob `**/entities/**/*.ts` or `**/entities/*.entity.ts`
- For Kysely: `**/migrations/**` + `**/db/**`
- For TypeORM: `ormconfig.*` + `**/entities/**`
- For Laravel: `database/migrations/*.php` (all of them) + `app/Models/*.php`
- For Knex/raw SQL: `**/migrations/**`
- Read ALL migration files to understand schema evolution and current state

**Messaging & events:**
- Grep for `RabbitMQ`, `amqp`, `rmq`, `@weroad/nestjs-rmq`, `golira-rmq`
- Find all consumers: grep for `@RabbitSubscribe`, `@MessagePattern`, `Consumer`, `consumer`
- Find all emitters/producers: grep for `publish`, `emit`, `RabbitPublisher`, `Emitter`
- Map exchange names, routing keys, payload shapes

**API surface:**
- For REST: grep `@Controller`, `@Get`, `@Post`, `@Put`, `@Delete`, `@Patch` — map all endpoints
- For GraphQL: grep `@Resolver`, `@Query`, `@Mutation`, `@Subscription` — map all operations
- For OpenAPI: read `openapi.ts`, `swagger.*`, any generated spec
- Read controller/resolver files to understand request/response shapes

**Inter-service dependencies:**
- Grep for HTTP clients: `axios`, `fetch`, `got`, `HttpService`, `@nestjs/axios`
- Find service URLs in env: `*_URL`, `*_HOST`, `*_API_URL`
- Find internal imports: `@weroad/*` packages
- Map which services this repo calls and which call it

**Auth & security:**
- Grep for `@Public`, `@CheckPermission`, `@Auth`, `Guard`, `FusionAuth`, JWT patterns
- Map auth strategy per endpoint group (public, M2M, user JWT, admin)

**Background jobs:**
- Grep for `@Cron`, `@Interval`, `cron`, `schedule`, `worker`
- Map schedule, purpose, health checks

**Testing:**
- Read test config: `vitest.config.*`, `jest.config.*`, `phpunit.xml`
- Identify test types available: unit, e2e, integration, eval

**Source structure:**
- `ls` the top-level dirs and `src/` (or equivalent) to map the module structure
- For NestJS: read `app.module.ts` to understand module wiring
- For monorepos: map each workspace and its role

### 2. Check existing L3 file

Read `outputs/services/{owner}-{repo}.AGENT.MD` if it exists. Compare against what you found.
Preserve any manually-added context (marked with comments or clearly editorial) unless
it's now wrong.

### 3. Check team ownership

Look up the repo in `memory/L1/entities.md` or the team L2 files to identify the owner team.
Also check CODEOWNERS if present.

## Output Format

Write to `outputs/services/{owner}-{repo}.AGENT.MD` using this structure. Every section that
has data MUST be included. Skip sections only if the repo genuinely doesn't have that
concept (e.g., no DB for a stateless service).

```markdown
# {owner}/{repo}

> {One-line description — what the service does in the WeRoad ecosystem}
**Source:** `src/github/{owner}/{repo}/`

<!-- verified: {today YYYY-MM-DD} | source: src/github/{owner}/{repo}/ -->

## Stack
{Bullet list: framework, language, runtime, DB, ORM, cache, messaging, auth, key libs, version}

## Source Structure
{ASCII tree of top-level dirs + key files with 1-line annotations}

## Architecture
{Paragraph explaining the architectural pattern (layered, DDD, MVC, etc.)}
{For monorepos: explain each workspace and how they relate}

## Database Schema
{Table of ALL models/entities with: name, table, key columns, relations}
{Entity relationship summary — which models reference which}
{Enum types with all values}
{Migration count and latest migration description}

## API Surface
{Table of all endpoints/operations: method, path/operation, auth, description}
{Group by domain/controller/resolver}
{Note request/response shapes for non-obvious endpoints}

## Request Flows
{For each major flow (2-5 flows): numbered steps showing the path through the code}
{Include: entry point → validation → business logic → DB → events → response}

## Messaging (RabbitMQ / Events)
### Produced Events
{Table: event name, routing key pattern, trigger, payload shape}

### Consumed Events
{Table: event name, routing key pattern, handler, what it does}

{Exchange and queue configuration}

## Inter-Service Dependencies
### This service calls:
{Table: service, protocol, purpose, env var for URL}

### Called by:
{List services that call this one, if discoverable from env/docs}

## Auth Patterns
{Map of auth strategies per endpoint group}
{Roles, permissions, scopes relevant to this service}

## Background Jobs
{Table: job name, schedule (cron expression), purpose, health check}

## Configuration
{Key env vars grouped by concern: server, DB, messaging, external services, feature flags}
{Per-market/country configuration if applicable}

## Testing
{Available test types, commands to run them, any special setup (Docker, etc.)}

## Key Files
{Table of the 10-15 most important files with purpose — the ones a developer needs first}

## Commands
{Code block with the most common dev commands: start, test, build, migrate, lint}

## Owner
{Team name}
```

## 4. Update L3/cross RabbitMQ Topology Files (if service has messaging)

If the service produces or consumes RabbitMQ events, you MUST also update the central
RabbitMQ documentation files in `outputs/services/cross/`:

### Files to update:
- `outputs/services/cross/weroad-rabbitmq-producers-consumers.md` — Add/update events in the reference table
- `outputs/services/cross/weroad-rabbitmq-schema.md` — Add/update payload schemas for new events
- `outputs/services/cross/weroad-rabbitmq-topology.md` — Update the mermaid diagram and matrices

### Process:

1. **Read all three L3/cross files** to understand the current structure and formatting
2. **For producers-consumers.md**: 
   - Add new events to the Event Reference Table with proper routing keys, producers, consumers, and descriptions
   - Update the Exchange Summary table with accurate producer/consumer counts
3. **For schema.md**:
   - Add new event schemas under the appropriate exchange section
   - Follow the existing JSON format with field types and descriptions
4. **For topology.md**:
   - Update the mermaid diagram to include new producer/consumer connections
   - Update the Producer and Consumer matrices
   - Update the Exchange Summary counts

### Rules for L3/cross updates:
- Preserve existing formatting and conventions
- Group events by exchange in the same order as existing sections
- Use `{cc}` placeholder for country code in routing keys (e.g., `{cc}.booking.created`)
- Mark unknown consumers/producers as `TBD` or `_source TBD_` — do not guess
- Update the `<!-- verified: -->` comment with today's date

## Rules

- Read the actual source code. Do not guess or infer from file names alone.
- For database schemas: list EVERY model and its columns. This is the most valuable
  part of the documentation — developers need to know what's in the DB without reading
  migration files. Include column types, nullable flags, defaults, and indexes for
  important tables.
- For messaging: capture the exact routing key patterns and payload interfaces.
  Messaging bugs are the hardest to debug — complete docs here save hours.
- **Always update L3/cross RabbitMQ files when the service has messaging** — keep the topology
  documentation in sync with the service-level documentation.
- For dependencies: be specific. Not "calls catalog API" but "calls api-catalog via
  GraphQL at `API_CATALOG_INTERNAL_URL` for travel data".
- Keep the file under 1000 lines. Compress tables, use abbreviations in table cells.
- Set today's date in the `<!-- verified: -->` comment.
- If updating an existing file, show a summary of what changed before writing.
