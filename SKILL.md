---
name: Code Symphonist
description: Orchestrates multi-file refactoring while preserving architectural integrity
version: 1.2.0
author: SMOUJBOT Skill Registry
tags: [refactoring, orchestration, architecture, transformation]
dependencies: [git, node>=18, python>=3.9, eslint, typescript, prettier]
requires_workspace: true
---

# Code Symphonist

A specialist skill for orchestrating complex, multi-file refactoring operations that maintain architectural integrity, dependency contracts, and behavioral consistency across an entire codebase.

## Purpose

Real use cases:

- **Schema Migration Orchestration**: When evolving a database schema in a Prisma/TypeORM project, coordinate updates across entity definitions, DTOs, validation layers, API contracts, and test fixtures in a single coherent operation with rollback capability.

- **Modular Extraction**: Split a 5000-line monolith service into feature modules while preserving all dependencies, ensuring no circular imports are introduced, and maintaining backward compatibility through adapter layers.

- **API Versioning Transition**: Migrate from REST v1 to GraphQL v2 by systematically transforming controllers, services, clients, and tests while keeping both versions operational during transition.

- **Framework Migration**: Upgrade from Angular 12 to 15 or React 17 to 18 with concurrent compatibility mode, coordinating changes across component lifecycle methods, state management, and build configuration.

- **Cross-Cutting Concern Refactoring**: Apply consistent logging, error handling, or authentication patterns across 100+ files while respecting each module's context.

## Scope

Commands (real implementations):

```bash
# Analyze architecture before refactoring
code-symphonist analyze --src src/ --arch arch-diagram.json --deps graph.dot

# Plan multi-file transformation
code-symphonist plan --target "extract user module" --strategy gradual --output plan.yaml

# Execute with dry-run first
code-symphonist execute --plan plan.yaml --dry-run --report changes.json

# Apply with backup and rollback capability
code-symphonist apply --plan plan.yaml --backup-dir .backup/$(date +%s) --commit-prefix "refactor: "

# Verify architectural integrity
code-symphonist verify --arch arch-diagram.json --test-integration --check-circular

# Generate rollback manifest
code-symphonist rollback --from-commit HEAD~3 --output rollback-manifest.json

# Batch transform with templates
code-symphonist transform --template templates/api-to-graphql.hbs --files "src/api/**/*.ts" --context context.yaml
```

## Work Process

**Phase 1: Discovery (15-30 min)**

1. Clone target repository to isolated workspace: `git clone --branch feat/refactor-orchestra <repo> /tmp/cs-workspace`
2. Run static analysis: `code-symphonist analyze --src /tmp/cs-workspace/src --output分析报告.json`
3. Generate dependency graph: `cd /tmp/cs-workspace && npx madge --image graph.svg src/`
4. Identify architectural boundaries: parse `分析报告.json` for module coupling metrics, circular dependencies, and layer violations
5. Catalog all affected file types: `.ts` services, `.proto` definitions, `.sql` migrations, test files, config files

**Phase 2: Planning (20-45 min)**

1. Create transformation plan YAML:
```yaml
id: "extract-auth-2024-03"
phases:
  - name: "split auth service"
    steps:
      - action: "create_module"
        path: "src/auth/"
        from: "src/services/authService.ts"
      - action: "extract_interface"
        from: "src/services/authService.ts"
        to: "src/auth/types.ts"
      - action: "update_imports"
        files: ["src/**/*.ts"]
        changes:
          - old: "'../services/authService'"
            new: "'../auth/authService'"
        # 47 files affected
```

2. Validate backward compatibility: simulate import resolution for all changed files
3. Generate test impact matrix: which unit/integration/e2e tests will break
4. Create rollback checklist: file hashes before changes, database migration undo scripts

**Phase 3: Execution (30-90 min)**

1. Create backup: `git add -A && git commit -m "backup: pre-refactor" && git tag refactor-backup-$(date +%Y%m%d-%H%M%S)`
2. For each phase step:
   - Run transformation with template engine: `handlebars templates/service.hbs -d context.yaml > newfile.ts`
   - Apply codemod: `jscodeshift -t transforms/import-rename.js src/**/*.ts`
   - Run AST-based verification: `ts-node scripts/verify-ast.js --file newfile.ts --schema schema.json`
3. After each phase run: `npm run build && npm run typecheck`
4. On failure: stop, analyze error, adjust plan, retry from last backup point

**Phase 4: Verification (15-25 min)**

1. Run architectural integrity check:
```bash
code-symphonist verify \
  --arch arch-diagram.json \
  --before-commit refactor-backup-* \
  --after-commit HEAD \
  --check-circular \
  --check-coupling
```

2. Execute integration tests: `npm run test:integration -- --grep "Auth Flow"`
3. Smoke test development server: `npm run dev & sleep 5 && curl -f http://localhost:3000/health`
4. Generate verification report: `code-symphonist report --output refactor-report.md`

**Phase 5: Documentation**

1. Update ADRs (Architecture Decision Records)
2. Generate migration guide for team
3. Create rollback runbook

## Golden Rules

1. **Never refactor without a backup tag**: Every execution must create a git tag before first change.
2. **Preserve behavioral contracts**: Functional changes only if explicitly requested; refactoring must pass all existing tests.
3. **Maintain backward compatibility**: Public APIs (exports, function signatures) unchanged unless major version bump is approved.
4. **One logical change per commit**: Split large refactoring into atomic commits (function extraction → move file → update imports).
5. **AST over regex**: Use TypeScript compiler API for code transformation; never use sed/grep on source.
6. **Test after every phase**: Build + typecheck + unit tests before proceeding to next phase.
7. **Circular dependency guard**: Fail if new cycles introduced; use `depcruise` to enforce.
8. **Rollback within 5 minutes**: Ability to revert entire refactoring in under 5 minutes via `git reset --hard refactor-backup-*`.
9. **No uncommitted changes**: Workspace must be clean before starting.
10. **Document every transformation**: Log each file changed, reason, and verification status.

## Examples

**Example 1: Extract authentication service**

User prompt: "Extract auth logic from src/services/authService.ts into its own module. The authService has 800 lines, uses bcrypt, JWT, and has dependencies on User model and Mailer. Create src/auth/ with proper exports."

Input to Code Symphonist:
```
{
  "operation": "module_extraction",
  "source_file": "src/services/authService.ts",
  "target_module": "src/auth/",
  "dependencies": ["src/models/User.ts", "src/utils/mailer.ts"],
  "preserve_exports": true,
  "create_index": true,
  "update_references": ["src/**/*.ts", "tests/**/*.ts"]
}
```

Output plan.yaml phase:
```yaml
steps:
  - action: "parse_ast"
    file: "src/services/authService.ts"
    output: "ast.json"
  - action: "extract_types"
    from_ast: "ast.json"
    to: "src/auth/types.ts"
  - action: "create_service"
    from_ast: "ast.json"
    to: "src/auth/authService.ts"
    keep_in_place: false
  - action: "update_imports"
    files_glob: "src/**/*.ts"
    replacements:
      - pattern: "from '../services/authService'"
        replacement: "from '../auth/authService'"
      - pattern: "from '../../services/authService'"
        replacement: "from '../auth/authService'"
    # Verified: 23 import statements across 7 files
```

**Example 2: Convert REST endpoints to GraphQL resolvers**

User prompt: "Convert user API in src/controllers/userController.ts and src/routes/user.ts to GraphQL. Keep both REST and GraphQL operational during transition. Need types in src/graphql/types/."

Input:
```bash
code-symphonist plan \
  --target "dual API support" \
  --constraints "REST deprecated=false, GraphQL version=v2" \
  --output dual-api-plan.yaml
```

Generated steps include:
- Parse existing REST routes (Express)
- Generate SDL from Joi validation schemas
- Create resolver stubs matching controller methods
- Add feature flag `features.enableGraphQL`
- Update clients to use GraphQL when flag enabled

**Example 3: Apply consistent error handling**

User prompt: "Add standardized error handling across all service files. All functions should return { success: boolean, data?: T, error?: { code: string, message: string } }."

Transformation template `templates/error-handling.hbs`:
```handlebars
{{#each functions}}
async function {{name}}({{params}}) {
  try {
    const result = await originalImplementation({{params}});
    return { success: true, data: result };
  } catch (err) {
    logger.error('{{name}} failed', err);
    return { 
      success: false, 
      error: { 
        code: '{{module}}.{{name}}_FAILED',
        message: err.message 
      } 
    };
  }
}
{{/each}}
```

Applied to 87 service files via: `code-symphonist transform --template error-handling.hbs --files "src/services/**/*.ts" --dry-run`

## Rollback Commands

**Complete rollback** (if refactoring incomplete or broken):
```bash
# Find backup tag
git tag --list "refactor-backup-*" | sort -r | head -1
# Example output: refactor-backup-20240315-143022

# Reset entire workspace
git reset --hard refactor-backup-20240315-143022
git clean -fd  # Remove new files
```

**Partial rollback** (phase 2 failed, keep phase 1):
```bash
# View commit history to find phase boundaries
git log --oneline --grep "refactor:"

# Reset to specific phase
git reset --hard HEAD~3  # Go back 3 commits to phase 1 end
```

**File-level rollback** (single file corrupted):
```bash
# Compare file with backup
git show refactor-backup-20240315-143022:src/auth/authService.ts > /tmp/authService.backup.ts

# Restore specific file
git checkout refactor-backup-20240315-143022 -- src/auth/authService.ts
```

**Database schema rollback** (if migration included):
```bash
# Use generated migration undo script
psql -U user -d db -f migrations/003_add_auth_tables_down.sql

# Or use Prisma migrate
npx prisma migrate resolve --rolled-back 003_add_auth_tables
```

**Verify rollback success**:
```bash
npm run build
npm run typecheck
npm run test:unit -- --grep "auth"
# All tests must pass
```

## Dependencies and Requirements

- **Git 2.30+**: For tagging, branching, and atomic commits
- **Node.js 18+**: For executing transforms and TypeScript AST operations
- **Python 3.9+**: For graph analysis and dependency calculations
- **TypeScript Compiler API**: `npm install -D typescript ts-morph`
- **jscodeshift**: `npm install -g jscodeshift`
- **madge**: `npm install -g madge` (circular dependency detection)
- **depcruise**: `npm install -g dependency-cruiser` (architectural boundary enforcement)
- **handlebars**: `npm install -g handlebars` (template rendering)

Environment variables:
```bash
# Enable verbose logging
export CS_DEBUG=1
# Set backup directory override
export CS_BACKUP_DIR="/mnt/backups/refactors"
# Slack/Teams notification webhook for completion
export CS_NOTIFY_WEBHOOK="https://hooks.slack.com/services/..."
```

## Verification Steps

1. **Build verification**: `npm run build` exits 0
2. **TypeScript check**: `npx tsc --noEmit` exits 0
3. **Import integrity**: `npx ts-unused-exports src/` reports no missing exports
4. **Circular deps**: `depcruise src/ --circular --ignore-pattern "**/node_modules/**"` exits 0
5. **Architecture compliance**: `code-symphonist verify --arch arch-diagram.json --fail-on-violation`
6. **Integration tests**: `npm run test:integration` all pass
7. **Smoke test**: `curl -f http://localhost:3000/health` returns 200

## Troubleshooting

**Issue**: "Circular dependency introduced between module A and B"
```
Diagnosis: depcruise detected new cycle after phase 3
Action: 
1. git diff refactor-backup-* src/moduleA.ts src/moduleB.ts
2. Identify import that created cycle
3. Introduce interface in shared/ folder
4. Update plan.yaml to add "extract_interface" step before phase 3
5. code-symphonist apply --plan adjusted-plan.yaml --from-phase 2
```

**Issue**: "Build fails with 'Cannot find module' after refactoring"
```
Diagnosis: Import paths not updated correctly for nested directory changes
Action:
1. code-symphonist analyze-deps --before refactor-backup-* --after HEAD
2. Identify mismatched paths
3. Edit plan.yaml update_imports step with additional patterns
4. Re-run: code-symphonist apply --plan plan.yaml --from-step "update_imports"
```

**Issue**: "Tests fail due to changed mock locations"
```
Diagnosis: Test fixtures not moved alongside source files
Action:
1. Locate test mocks: find src/**/__mocks__/ or tests/__fixtures__
2. Add mirror structure to target module
3. Update jest.config.js moduleNameMapper if paths changed
4. Rerun affected tests: npm run test:unit -- --findRelatedTests src/auth/authService.ts
```

**Issue**: "Transformation template produces invalid TypeScript"
```
Diagnosis: Handlebars template missing generic type parameters or async keyword
Action:
1. Validate generated file: npx @typescript-eslint/parser < generated.ts
2. Add @typescript-eslintparser directives to template
3. Use ts-morph instead of Handlebars for type-aware transformations
4. Update dependency: npm install -D ts-morph
```

```