# Tooling & Developer Setup

## Required Tools

| Tool | Version | Purpose |
|---|---|---|
| Node.js | 22.x | Runtime |
| Angular CLI | 20.x | Scaffolding, build |
| npm | 10.x | Package manager |

## Angular CLI Commands

```bash
# Create standalone component (default in Angular 17+)
ng generate component pages/feature/feature-name

# Create service
ng generate service core/services/feature-name

# Create guard
ng generate guard core/guards/feature-name

# Create pipe
ng generate pipe core/pipes/feature-name

# Create directive
ng generate directive core/directives/feature-name

# Build environments
npm run build.dev
npm run build.tst
npm run build.hml
npm run build.prd
```

## Code Quality

### ESLint

```bash
npx eslint src/ --fix
```

Config: `.eslintrc.json`

### Prettier

```bash
npx prettier src/ --write
```

Config: `.prettierrc`  
Print width: 100 | Single quotes | Angular HTML parser for templates

### Lint-Staged (on commit)

Configured in `.lintstagedrc.json` — runs ESLint + Prettier on staged files automatically.

## Testing

### Unit Tests (Karma + Jasmine)

```bash
npm test
```

### E2E Tests (Cypress)

```bash
npm run cypress:open        # Interactive mode
npm run cypress:run         # Headless mode (CI)
npm run cypress:run:chrome  # Chrome browser
npm run e2e                 # Start dev server + run tests
```

## Build Environments

This project supports **multi-client white-labeling** via Angular build configurations.

| Script | Environment | Client |
|---|---|---|
| `build.dev` | dev | default |
| `build.prd` | production | default |
| `build.dev-farmadez` | dev | farmadez |
| `build.prd-farmadez` | production | farmadez |
| `build.dev-venancio` | dev | venancio |
| `build.prd-venancio` | production | venancio |

Client assets are in `src/assets/client-assets/` and `client-configs/`.

## CI/CD

Azure Pipelines — config in `azure-pipelines.yml` and `azure-pipelines/` folder.

## Docker

Dockerfiles available in `dockerfiles/` for containerized deployments.

## Multi-Client Configuration

```bash
# Configure assets for a specific client
npm run configure:client
```

This runs `tools/configure-client.mjs` to symlink/copy client-specific assets.
