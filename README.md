# Git Hooks para Revisión de Código con IA

Este directorio contiene hooks de Git para realizar revisiones automáticas de código usando la API de Gemini.

## Archivos Disponibles

### 1. `post-commit` (Existente)
Hook completamente independiente en Bash que replica la funcionalidad del Lambda.

**Características:**
- ✅ Completamente autónomo (solo requiere `bash`, `curl`, y opcionalmente `jq`)
- ✅ Usa el mismo prompt y lógica que el Lambda
- ✅ Template de prompt en archivo externo (`prompt_template.txt`)
- ✅ **Detección inteligente de rama objetivo** para git diff
- ✅ Configurable directamente en el script
- ✅ Filtra archivos irrelevantes (`*enum*`, `*constant*`, `*.yml`)
- ✅ Respeta el límite mínimo de cambios (15 líneas por defecto)
- ✅ Detecta automáticamente cuando no hay problemas
- ✅ Guarda resultados en archivos temporales
- ✅ Funciona sin `jq` (con extracción JSON nativa)

### 3. `prompt_template.txt` (Nuevo)
Archivo de plantilla que contiene el prompt completo utilizado por el hook standalone. Este archivo permite:
- ✅ Editar el prompt sin modificar el código del script
- ✅ Mantener sincronizado el prompt con el Lambda
- ✅ Personalizar lineamientos REST y Swagger según el proyecto

## Configuración del Hook Standalone

### 1. Instalar el Hook

Se deben mover el post-commit y el template a su carpeta de hooks

### 2. Configurar la API Key de Gemini

Edita el archivo `.git/hooks/post-commit` y reemplaza:

```bash
GEMINI_API_KEY="your-gemini-api-key-here"
```

Con tu clave real de la API de Gemini.

### 3. Configuración Opcional

Puedes modificar estas variables en el script:

```bash
# Mínimo número de cambios requeridos para ejecutar la revisión
MIN_CHANGES=1
```

### 4. Desactivar Temporalmente

Para saltar la revisión en un commit específico:

```bash
SKIP_AI_REVIEW=1 git commit -m "commit sin revisión"
```

## Dependencias

### Requeridas:
- `bash` (disponible en todos los sistemas Unix/Linux/WSL)
- `curl` (para llamadas a la API de Gemini)
- `git` (obviamente)

### Opcionales:
- `jq` (para mejor formateo de respuestas JSON)

## Detección Inteligente de Rama Objetivo

El hook utiliza **detección automática de la rama objetivo** basada en el nombre de la rama actual:

| Patrón en nombre de rama | Rama objetivo | Ejemplo |
|-------------------------|---------------|---------|
| `*dev*` o `*development*` | `development` | `ccamargo/DEV/CDIA-55` |
| `*ci*` o `*master*` | `master` | `feature/CI/nueva-validacion` |
| `*release*`, `*prod*` o `*production*` | `release` | `hotfix/RELEASE/bug-fix` |
| Ningún patrón | Commit padre | `random-branch-name` |

**Ventajas:**
- ✅ Revisa **todos los cambios** de tu feature vs la rama objetivo
- ✅ No solo el último commit, sino toda la diferencia acumulada
- ✅ Fallback automático si la rama objetivo no existe
- ✅ Case-insensitive (funciona con mayúsculas y minúsculas)

## Funcionamiento

El hook se ejecuta automáticamente después de cada commit y:

1. **Detecta** la rama objetivo basada en el nombre de la rama actual
2. **Verifica** que hay suficientes cambios para justificar una revisión
3. **Genera** el diff contra la rama objetivo (o commit padre como fallback)
4. **Filtra** solo archivos relevantes (excluye enums, constants, yml)
5. **Carga** el template de prompt desde archivo externo
6. **Envía** el diff a Gemini usando el mismo prompt que el Lambda
7. **Extrae** la respuesta JSON (con o sin `jq`)
8. **Muestra** los resultados en la terminal
9. **Guarda** los resultados en `commit_review_YYYYMMDD_HHMMSS.md` en la raíz del repositorio

## Ejemplo de Salida

```
=== Post-commit AI Code Review ===
Reviewing commit: a1b2c3d4e5f6g7h8i9j0
Commit message: Add new user validation logic
Comparing against target branch: development
Found 23 added lines
Filtering relevant files...
Including file in review: src/main/java/UserValidator.java
Loading prompt template...
Calling Gemini API for code review...

=== AI Code Review Results ===
# Resumen y propósito de la PR
- Se añade nueva lógica de validación de usuarios con verificación de email y teléfono

# Análisis de los cambios
## Posibles Bugs
- _No se encontraron problemas_

## Posibles problemas de Diseño / SOLID
- [UserValidator.java:15] Método validateUser hace demasiadas cosas – Considerar separar validaciones

## Posibles problemas de Clean Code
- [UserValidator.java:22] Variable 'isValid' podría tener nombre más descriptivo
- [UserValidator.java:18] Número mágico 10 – Considerar usar constante para longitud mínima

## Posibles problemas por impacto del Código Eliminado
- _No se encontraron problemas_

## Violaciones de lineamientos REST
- _No se encontraron problemas_

## Violaciones de lineamientos de Swagger
- _No se encontraron problemas_

Review saved to: /path/to/your/repo/commit_review_20240923_143022.md

⚠️  Issues found in the code review. Please review the feedback above.
=== Post-commit review completed ===
```

## Solución de Problemas

### Error: "curl command not found"
Instalar curl:
```bash
# Ubuntu/Debian/WSL
sudo apt-get install curl

# CentOS/RHEL
sudo yum install curl

# macOS
brew install curl
```

### Error: "Failed to call Gemini API"
- Verifica que tu API key sea válida
- Confirma que tienes conexión a internet
- Revisa que la API key tenga los permisos correctos

### El hook no se ejecuta
- Verifica que el archivo tiene permisos de ejecución: `ls -la .git/hooks/post-commit`
- Confirma que el archivo está en la ubicación correcta: `.git/hooks/post-commit`

### Warning: "jq is not installed"
El script funciona perfectamente sin `jq`, pero para mejor procesamiento de JSON puedes instalarlo:
```bash
# Ubuntu/Debian/WSL
sudo apt-get install jq

# CentOS/RHEL
sudo yum install jq

# macOS
brew install jq
```

### Error: "Prompt template file not found"
- Asegúrate de haber copiado `prompt_template.txt` junto con el hook
- Verifica que el archivo esté en el mismo directorio que el script
- El script busca el template en: `.git/hooks/prompt_template.txt`

## Comparación con el Lambda Original

| Característica | Lambda | Hook Standalone |
|---------------|---------|-----------------|
| Lenguaje | Python | Bash |
| Dependencias | Múltiples (boto3, google-genai, etc.) | Mínimas (curl, opcionalmente jq) |
| Configuración | Variable entorno/archivo | Directa en script + template externo |
| Prompt | Archivo Python | Archivo externo `prompt_template.txt` |
| Filtrado de archivos | Idéntico | Idéntico |
| Límite mínimo cambios | Idéntico | Idéntico |
| Detección "sin problemas" | Idéntico | Idéntico |
| Comparación Git | Commits específicos de PR | **Detección inteligente de rama objetivo** |
| Integración | AWS CodeCommit | Git local |
| Portabilidad | Específica AWS | Universal |
| Extracción JSON | Biblioteca Python | Nativo bash (con fallback sin jq) |
