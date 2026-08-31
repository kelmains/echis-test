# OHS Monorepo Template

A polyglot monorepo template for Open Health Stack (OHS) / FHIR-based projects.
It gives specialized teams (Mobile, Web, Backend, DevOps) independent release
pipelines while keeping a single source of truth for health data schemas.

See `Cross-Project_Automation.md`-style rationale in `docs/architecture.md`
for the full justification (schema flux, team autonomy, external consumption,
infra alignment).

## What's inside

```
.github/            CI/CD workflows + CODEOWNERS (path-filtered per team)
apps/                Team workspaces: mobile-kotlin, web-react, core-platform
packages/            shared-libraries: FHIR JSON Schemas -> generated TS + Kotlin
infra/               Terraform, Ansible, Docker security baselines
docs/                Architecture decisions and onboarding docs
```

## How to instantiate this template for a new project

1. **Clone / use as template**
   ```bash
   git clone <this-repo-url> my-project-repo
   cd my-project-repo
   rm -rf .git && git init
   ```

2. **Rename the workspace root**
   - `package.json` -> update `"name"` to `"@<project>/workspace-root"`
   - `settings.gradle.kts` -> update `rootProject.name`

3. **Update CODEOWNERS**
   - Edit `.github/CODEOWNERS` and replace the placeholder team handles
     (`@org/mobile-team`, `@org/web-team`, `@org/backend-team`, `@org/devops-team`)
     with your actual GitHub team handles.

4. **Add your FHIR schemas**
   - Drop project-specific JSON Schemas into `packages/shared-libraries/schemas/`
   - Run the generator:
     ```bash
     node packages/shared-libraries/generate.js
     ```
   - This populates `generated-ts/` and `generated-kotlin/`, consumed by
     `apps/web-react`, `apps/core-platform`, and `apps/mobile-kotlin`.

5. **Configure infra**
   - `infra/terraform/environments/dev.tfvars` and `prod.tfvars` — fill in
     your cloud account/project IDs.
   - `infra/ansible/inventory/` — add your staging/production host inventories.

6. **Set up environment variables**
   One shared `.env.example` at the repo root covers every service — copy
   it to `.env` (gitignored):
   ```bash
   cp .env.example .env
   ```
   Each service reads from this single file:
   - `apps/web-react` — via `envDir` in `vite.config.ts` (points at repo root)
   - `apps/core-platform/gateway-node` — via `dotenv` with an explicit path
     to the root `.env`
   - `apps/core-platform` docker-compose services — via
     `docker compose --env-file ../../.env`
   - `infra/terraform` and `packages/shared-libraries` (local/manual use
     only) — `source .env` before running commands by hand

   Never commit a real `.env` — only `.env.example` is tracked.

7. **Local dev bootstrap**
   ```bash
   cd apps/core-platform
   docker compose -f docker-compose.local.yml up
   ```
   This spins up local FHIR server(s) + databases so all teams can develop
   against a consistent backend.

8. **Verify path-filtered CI**
   - Push a change under `apps/web-react/` only — confirm only `cd-web.yml`
     fires. Repeat for `apps/mobile-kotlin/`, `apps/core-platform/`, and
     `infra/` to confirm blast-radius isolation is working.

## Team ownership model

| Path | Owning team |
|---|---|
| `apps/mobile-kotlin/` | Mobile |
| `apps/web-react/` | Web |
| `apps/core-platform/` | Backend |
| `infra/` | DevOps |
| `packages/shared-libraries/schemas/` | Cross-team (requires 2 approvals) |

## Evolution strategy

This monorepo is designed as a one-way door that's easy to exit: each
top-level directory under `apps/` and `infra/` is intentionally
self-contained so it can be split into its own repository later with
minimal history rewriting, if/when team scale demands it.

(Git test- please ignore -)
