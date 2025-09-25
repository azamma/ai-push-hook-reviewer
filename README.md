# pre-push para Revisión de Código con IA

Este directorio contiene un hook `pre-push` de Git para realizar revisiones automáticas de código usando la API de Gemini.

## Archivos Disponibles

### 1. `pre-push`
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
- 🛑 **Bloqueo de push** si se detectan problemas

### 2. `prompt_template.txt`
Archivo de plantilla que contiene el prompt completo utilizado por el hook. Este archivo permite:
- ✅ Editar el prompt sin modificar el código del script
- ✅ Mantener sincronizado el prompt con el Lambda
- ✅ Personalizar lineamientos REST y Swagger según el proyecto

## Configuración del Hook

### 1. Instalar el Hook

Se deben mover el `pre-push` y el `prompt_template.txt` a su carpeta de hooks: `.git/hooks/`

### 2. Configurar la API Key de Gemini

Edita el archivo `.git/hooks/pre-push` y reemplaza:

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

Para saltar la revisión en un push específico:

```bash
SKIP_AI_REVIEW=1 git push
```

## Bloqueo de Push

Si la revisión de la IA encuentra algún problema en las siguientes categorías, **el push será detenido**:
- `Posibles Bugs`
- `Posibles problemas de Diseño / SOLID`
- `Posibles problemas de Clean Code`
- `Violaciones de lineamientos REST`
- `Violaciones de lineamientos de Swagger`

El script revisa si estas secciones en la respuesta de la IA contienen algo diferente a "_No se encontraron problemas_". Si se genera un "Prompt AI para corregir errores detectados", el push también se detendrá.

Para forzar el push, puedes usar la variable de entorno `SKIP_AI_REVIEW=1`.

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

El hook se ejecuta automáticamente antes de cada `push` y:

1. **Detecta** la rama objetivo basada en el nombre de la rama actual.
2. **Verifica** que hay suficientes cambios para justificar una revisión.
3. **Genera** el diff contra la rama objetivo.
4. **Filtra** solo archivos relevantes.
5. **Carga** el template de prompt desde `prompt_template.txt`.
6. **Envía** el diff a Gemini.
7. **Extrae** y **analiza** la respuesta en busca de problemas.
8. **Muestra** los resultados en la terminal.
9. **Guarda** los resultados en un archivo.
10. **Bloquea el `push`** si se encontraron problemas, saliendo con un código de error.

## Ejemplo de Salida

```
=== Pre-push AI Code Review ===
Reviewing changes before push...
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
=== Pre-push review completed ===

[pre-push hook] *** PUSH REJECTED BY AI CODE REVIEW ***
[pre-push hook] Issues found. Please fix them or use SKIP_AI_REVIEW=1 to bypass.
error: failed to push some refs to '...'
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
- Verifica que tu API key sea válida.
- Confirma que tienes conexión a internet.
- Revisa que la API key tenga los permisos correctos.

### El hook no se ejecuta
- Verifica que el archivo tiene permisos de ejecución: `ls -la .git/hooks/pre-push`
- Confirma que el archivo está en la ubicación correcta: `.git/hooks/pre-push`

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
- Asegúrate de haber copiado `prompt_template.txt` junto con el hook.
- Verifica que el archivo esté en el mismo directorio que el script: `.git/hooks/prompt_template.txt`

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
| Integración | AWS CodeCommit | Git local (`pre-push`) |
| Portabilidad | Específica AWS | Universal |
| Extracción JSON | Biblioteca Python | Nativo bash (con fallback sin jq) |

