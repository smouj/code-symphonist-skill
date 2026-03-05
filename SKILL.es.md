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

Un skill especializado para orquestar operaciones complejas de refactoring en múltiples archivos que mantienen la integridad arquitectónica, los contratos de dependencia y la consistencia de comportamiento en toda la codebase.

## Propósito

Casos de uso reales:

- **Orquestación de Migración de Schema**: Al evolucionar un schema de base de datos en un proyecto Prisma/TypeORM, coordinar actualizaciones a través de definiciones de entidades, DTOs, capas de validación, contratos de API y fixtures de pruebas en una única operación coherente con capacidad de rollback.

- **Extracción Modular**: Dividir un servicio monolítico de 5000 líneas en módulos de características preservando todas las dependencias, asegurando que no se introduzcan imports circulares y manteniendo compatibilidad hacia atrás a través de capas adaptadoras.

- **Transición de Versionado de API**: Migrar de REST v1 a GraphQL v2 transformando sistemáticamente controladores, servicios, clientes y pruebas manteniendo ambas versiones operativas durante la transición.

- **Migración de Framework**: Actualizar de Angular 12 a 15 o React 17 a 18 con modo de compatibilidad concurrente, coordinando cambios en métodos de ciclo de vida de componentes, gestión de estado y configuración de build.

- **Refactoring de Preocupaciones Transversales**: Aplicar patrones consistentes de logging, manejo de errores o autenticación en 100+ archivos respetando el contexto de cada módulo.

## Alcance

Comandos (implementaciones reales):

```bash
# Analizar arquitectura antes del refactoring
code-symphonist analyze --src src/ --arch arch-diagram.json --deps graph.dot

# Planificar transformación multi-archivo
code-symphonist plan --target "extract user module" --strategy gradual --output plan.yaml

# Ejecutar con dry-run primero
code-symphonist execute --plan plan.yaml --dry-run --report changes.json

# Aplicar con backup y capacidad de rollback
code-symphonist apply --plan plan.yaml --backup-dir .backup/$(date +%s) --commit-prefix "refactor: "

# Verificar integridad arquitectónica
code-symphonist verify --arch arch-diagram.json --test-integration --check-circular

# Generar manifiesto de rollback
code-symphonist rollback --from-commit HEAD~3 --output rollback-manifest.json

# Transformar por lotes con plantillas
code-symphonist transform --template templates/api-to-graphql.hbs --files "src/api/**/*.ts" --context context.yaml
```

## Proceso de Trabajo

**Fase 1: Descubrimiento (15-30 min)**

1. Clonar repositorio objetivo en workspace aislado: `git clone --branch feat/refactor-orchestra <repo> /tmp/cs-workspace`
2. Ejecutar análisis estático: `code-symphonist analyze --src /tmp/cs-workspace/src --output分析报告.json`
3. Generar gráfico de dependencias: `cd /tmp/cs-workspace && npx madge --image graph.svg src/`
4. Identificar límites arquitectónicos: parsear `分析报告.json` para métricas de acoplamiento de módulos, dependencias circulares y violaciones de capas
5. Catalogar todos los tipos de archivo afectados: servicios `.ts`, definiciones `.proto`, migraciones `.sql`, archivos de prueba, archivos de configuración

**Fase 2: Planificación (20-45 min)**

1. Crear YAML de plan de transformación:
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
        # 47 archivos afectados
```

2. Validar compatibilidad hacia atrás: simular resolución de imports para todos los archivos modificados
3. Generar matriz de impacto en pruebas: qué pruebas unitarias/integración/e2e se verán afectadas
4. Crear checklist de rollback: hashes de archivos antes de cambios, scripts de deshacer para migraciones de base de datos

**Fase 3: Ejecución (30-90 min)**

1. Crear backup: `git add -A && git commit -m "backup: pre-refactor" && git tag refactor-backup-$(date +%Y%m%d-%H%M%S)`
2. Para cada paso de fase:
   - Ejecutar transformación con motor de plantillas: `handlebars templates/service.hbs -d context.yaml > newfile.ts`
   - Aplicar codemod: `jscodeshift -t transforms/import-rename.js src/**/*.ts`
   - Ejecutar verificación basada en AST: `ts-node scripts/verify-ast.js --file newfile.ts --schema schema.json`
3. Después de cada fase ejecutar: `npm run build && npm run typecheck`
4. En fallo: detener, analizar error, ajustar plan, reintentar desde último punto de backup

**Fase 4: Verificación (15-25 min)**

1. Ejecutar verificación de integridad arquitectónica:
```bash
code-symphonist verify \
  --arch arch-diagram.json \
  --before-commit refactor-backup-* \
  --after-commit HEAD \
  --check-circular \
  --check-coupling
```

2. Ejecutar pruebas de integración: `npm run test:integration -- --grep "Auth Flow"`
3. Smoke test del servidor de desarrollo: `npm run dev & sleep 5 && curl -f http://localhost:3000/health`
4. Generar reporte de verificación: `code-symphonist report --output refactor-report.md`

**Fase 5: Documentación**

1. Actualizar ADRs (Architecture Decision Records)
2. Generar guía de migración para el equipo
3. Crear runbook de rollback

## Reglas de Oro

1. **Nunca refactorizar sin un tag de backup**: Cada ejecución debe crear un git tag antes del primer cambio.
2. **Preservar contratos de comportamiento**: Cambios funcionales solo si se solicitan explícitamente; el refactoring debe pasar todas las pruebas existentes.
3. **Mantener compatibilidad hacia atrás**: APIs públicas (exports, signatures de funciones) sin cambios a menos que se apruebe un major version bump.
4. **Un cambio lógico por commit**: Dividir refactoring grande en commits atómicos (extracción de función → mover archivo → actualizar imports).
5. **AST sobre regex**: Usar TypeScript compiler API para transformación de código; nunca usar sed/grep en código fuente.
6. **Probar después de cada fase**: Build + typecheck + pruebas unitarias antes de proceder a siguiente fase.
7. **Guardian de dependencias circulares**: Fallar si se introducen nuevos ciclos; usar `depcruise` para enforce.
8. **Rollback en menos de 5 minutos**: Capacidad de revertir refactoring completo en menos de 5 minutos via `git reset --hard refactor-backup-*`.
9. **Sin cambios sin commit**: Workspace debe estar limpio antes de comenzar.
10. **Documentar cada transformación**: Registrar cada archivo cambiado, razón y estado de verificación.

## Ejemplos

**Ejemplo 1: Extraer servicio de autenticación**

Prompt del usuario: "Extract auth logic from src/services/authService.ts into its own module. The authService has 800 lines, uses bcrypt, JWT, and has dependencies on User model and Mailer. Create src/auth/ with proper exports."

Input a Code Symphonist:
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

Fase plan.yaml generada:
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

**Ejemplo 2: Convertir endpoints REST a resolvers GraphQL**

Prompt del usuario: "Convert user API in src/controllers/userController.ts and src/routes/user.ts to GraphQL. Keep both REST and GraphQL operational during transition. Need types in src/graphql/types/."

Input:
```bash
code-symphonist plan \
  --target "dual API support" \
  --constraints "REST deprecated=false, GraphQL version=v2" \
  --output dual-api-plan.yaml
```

Pasos generados incluyen:
- Parsear rutas REST existentes (Express)
- Generar SDL desde schemas de validación Joi
- Crear stubs de resolvers que coincidan con métodos del controlador
- Añadir feature flag `features.enableGraphQL`
- Actualizar clientes para usar GraphQL cuando el flag esté habilitado

**Ejemplo 3: Aplicar manejo de errores consistente**

Prompt del usuario: "Add standardized error handling across all service files. All functions should return { success: boolean, data?: T, error?: { code: string, message: string } }."

Plantilla de transformación `templates/error-handling.hbs`:
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

Aplicada a 87 archivos de servicio via: `code-symphonist transform --template error-handling.hbs --files "src/services/**/*.ts" --dry-run`

## Comandos de Rollback

**Rollback completo** (si refactoring incompleto o roto):
```bash
# Encontrar tag de backup
git tag --list "refactor-backup-*" | sort -r | head -1
# Ejemplo de salida: refactor-backup-20240315-143022

# Resetear workspace completo
git reset --hard refactor-backup-20240315-143022
git clean -fd  # Eliminar archivos nuevos
```

**Rollback parcial** (fase 2 falló, mantener fase 1):
```bash
# Ver historial de commits para encontrar límites de fase
git log --oneline --grep "refactor:"

# Resetear a fase específica
git reset --hard HEAD~3  # Retroceder 3 commits al final de fase 1
```

**Rollback a nivel archivo** (archivo individual corrupto):
```bash
# Comparar archivo con backup
git show refactor-backup-20240315-143022:src/auth/authService.ts > /tmp/authService.backup.ts

# Restaurar archivo específico
git checkout refactor-backup-20240315-143022 -- src/auth/authService.ts
```

**Rollback de schema de base de datos** (si migración incluida):
```bash
# Usar script undo de migración generado
psql -U user -d db -f migrations/003_add_auth_tables_down.sql

# O usar Prisma migrate
npx prisma migrate resolve --rolled-back 003_add_auth_tables
```

**Verificar éxito de rollback**:
```bash
npm run build
npm run typecheck
npm run test:unit -- --grep "auth"
# Todas las pruebas deben pasar
```

## Dependencias y Requisitos

- **Git 2.30+**: Para tagging, branching y commits atómicos
- **Node.js 18+**: Para ejecutar transformaciones y operaciones AST de TypeScript
- **Python 3.9+**: Para análisis de gráficos y cálculos de dependencias
- **TypeScript Compiler API**: `npm install -D typescript ts-morph`
- **jscodeshift**: `npm install -g jscodeshift`
- **madge**: `npm install -g madge` (detección de dependencias circulares)
- **depcruise**: `npm install -g dependency-cruiser` (enforcement de límites arquitectónicos)
- **handlebars**: `npm install -g handlebars` (renderizado de plantillas)

Variables de entorno:
```bash
# Habilitar logging verbose
export CS_DEBUG=1
# Sets backup directory override
export CS_BACKUP_DIR="/mnt/backups/refactors"
# Slack/Teams notification webhook for completion
export CS_NOTIFY_WEBHOOK="https://hooks.slack.com/services/..."
```

## Pasos de Verificación

1. **Verificación de build**: `npm run build` sale con 0
2. **Check de TypeScript**: `npx tsc --noEmit` sale con 0
3. **Integridad de imports**: `npx ts-unused-exports src/` reporta sin exports faltantes
4. **Dependencias circulares**: `depcruise src/ --circular --ignore-pattern "**/node_modules/**"` sale con 0
5. **Cumplimiento arquitectónico**: `code-symphonist verify --arch arch-diagram.json --fail-on-violation`
6. **Pruebas de integración**: `npm run test:integration` todas pasan
7. **Smoke test**: `curl -f http://localhost:3000/health` retorna 200

## Solución de Problemas

**Problema**: "Circular dependency introduced between module A and B"
```
Diagnóstico: depcruise detectó nuevo ciclo después de fase 3
Acción: 
1. git diff refactor-backup-* src/moduleA.ts src/moduleB.ts
2. Identificar import que creó ciclo
3. Introducir interfaz en carpeta shared/
4. Actualizar plan.yaml para añadir paso "extract_interface" antes de fase 3
5. code-symphonist apply --plan adjusted-plan.yaml --from-phase 2
```

**Problema**: "Build fails with 'Cannot find module' after refactoring"
```
Diagnóstico: Import paths no actualizados correctamente para cambios de directorio anidado
Acción:
1. code-symphonist analyze-deps --before refactor-backup-* --after HEAD
2. Identificar paths mal coincidentes
3. Editar plan.yaml update_imports step con patrones adicionales
4. Re-ejecutar: code-symphonist apply --plan plan.yaml --from-step "update_imports"
```

**Problema**: "Tests fail due to changed mock locations"
```
Diagnóstico: fixtures de prueba no movidas junto con archivos fuente
Acción:
1. Localizar mocks de prueba: find src/**/__mocks__/ or tests/__fixtures__
2. Añadir estructura espejo al módulo objetivo
3. Actualizar jest.config.js moduleNameMapper si paths cambiaron
4. Re-ejecutar pruebas afectadas: npm run test:unit -- --findRelatedTests src/auth/authService.ts
```

**Problema**: "Transformation template produces invalid TypeScript"
```
Diagnóstico: Plantilla Handlebars faltando parámetros de tipo genéricos o keyword async
Acción:
1. Validar archivo generado: npx @typescript-eslint/parser < generated.ts
2. Añadir directivas @typescript-eslintparser a plantilla
3. Usar ts-morph en vez de Handlebars para transformaciones type-aware
4. Actualizar dependencia: npm install -D ts-morph
```

```