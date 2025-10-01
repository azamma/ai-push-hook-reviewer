# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an AI-powered Git hooks system for automated code review and PR documentation generation using the Gemini API. The system consists of two main bash scripts that integrate into the Git workflow:

- **pre-push**: Performs AI-driven code review before pushing changes, blocking pushes that contain critical issues
- **pr-details**: Generates comprehensive PR documentation automatically after a successful push

Both scripts are designed for Spring Boot microservices development with emphasis on REST API standards, SOLID principles, and clean code practices.

## Setup and Installation

### Initial Configuration

1. Copy files to `.git/hooks/` directory of target repository:
```bash
cp pre-push .git/hooks/
cp -r pre-push-resources .git/hooks/
```

2. Make scripts executable:
```bash
chmod +x .git/hooks/pre-push
chmod +x .git/hooks/pre-push-resources/pr-details
```

3. Configure API credentials in `pre-push-resources/gemini.ini`:
```ini
[gemini]
api_key = your-gemini-api-key-here
api_url = https://generativelanguage.googleapis.com/v1/models/gemini-2.5-pro:generateContent
```

### Directory Structure

```
.git/hooks/
├── pre-push                          # Main hook script
└── pre-push-resources/               # Supporting files
    ├── pr-details                    # PR details generator
    ├── gemini.ini                    # API configuration
    └── prompt_templates/             # AI prompt templates
        ├── review_prompt_template.txt
        └── pr_details_prompt_template.txt
```

### Dependencies

Required:
- `bash`
- `curl`
- `git`

Optional but recommended:
- `jq` - For reliable JSON parsing from Gemini API responses

## Key Commands

### Testing the Hook Locally

```bash
# Normal push (will trigger review)
git push

# Bypass AI review temporarily
SKIP_AI_REVIEW=1 git push
```

### Working with the Scripts

```bash
# Test pre-push hook manually
.git/hooks/pre-push

# Test pr-details generation manually
.git/hooks/pr-details
```

## Architecture and Workflow

### pre-push Hook Flow

1. **Dependency Check**: Validates `curl` and `jq` availability
2. **Target Branch Detection**: Automatically determines comparison branch based on current branch name patterns:
   - Branches with `dev`/`development` → compares against `development` branch
   - Branches with `ci`/`master` → compares against `master` branch
   - Branches with `release`/`prod`/`production` → compares against `release` branch
   - Otherwise falls back to parent commit
3. **Change Filtering**:
   - Requires minimum 15 added lines (configurable via `MIN_CHANGES` in pre-push:24)
   - Excludes files matching patterns: `*enum*`, `*constant*`, `*.yml`
4. **AI Review**: Sends filtered diff to Gemini API with review prompt template
5. **Issue Detection**: Parses response to check for critical issues in these sections:
   - Posibles Bugs
   - Posibles problemas de Diseño / SOLID
   - Posibles problemas de Clean Code
   - Violaciones de lineamientos REST
   - Violaciones de lineamientos de Swagger
6. **Push Control**: Blocks push if issues found (exit code 1), otherwise allows push
7. **PR Generation**: Always calls `pr-details` script after review completes

### pr-details Script Flow

1. Reads pushed refs from stdin (passed from pre-push hook)
2. Generates git diff for the push range
3. Sends diff to Gemini API with PR details prompt template
4. Saves generated documentation to `PR_DETAILS_<branch-name>_<timestamp>.md`

### Prompt Template System

Two separate prompt templates control AI behavior:

**review_prompt_template.txt** (Spanish):
- Acts as senior software engineer reviewer
- Analyzes only added/modified lines in diff
- Checks for: bugs, SOLID violations, clean code issues, deleted code impact, REST API guideline violations, Swagger documentation violations
- Returns structured markdown with specific sections
- Must return "_No se encontraron problemas_" for sections with no issues

**pr_details_prompt_template.txt** (Spanish):
- Acts as senior developer with 15+ years experience
- Generates professional PR documentation in markdown
- Includes: metadata table, executive summary, technical modifications, class index, test coverage, impact summary
- Maximum 4800 characters with automatic optimization
- Validates for typos in code and stops if found
- Suggests strategic entry point for code review

## Code Review Standards

The pre-push hook enforces these standards (defined in review_prompt_template.txt):

### REST API Guidelines (Lineamientos REST)
- Plural nouns for resources (e.g., `/customers` not `/customer`)
- Lowercase only, kebab-case for multi-word (e.g., `personal-info`)
- HTTP methods: GET (query), POST (create), PUT (replace), PATCH (partial update), DELETE (remove)
- API structure: `/{service-name}/{prefix}/{resource-path}`
- Prefixes: `b2c`, `b2b`, `bo`, `ext`, `iuse`, `sfc`, `notification`
- Versioning via `X-Api-Version` header

### Swagger/OpenAPI Standards (Lineamientos Swagger)
- Separate interface from controller implementation
- Use `@Tag` for grouping, `@Operation` for endpoint documentation
- Use `@Server` annotations to specify exposure (localhost, LB, APIGW)
- Use `@SecurityRequirement` for authentication documentation
- Use `@ErrorResponses` with custom `@ErrorResponse` annotations
- Use `@Schema` on models and fields with description, example, required

### SOLID Principles
- Single Responsibility, Open-Closed, Liskov Substitution, Interface Segregation, Dependency Inversion

### Clean Code
- Readability, clear naming, no duplication, no magic numbers, proper error messages, small functions

## Configuration Options

### pre-push Configuration (Lines 21-31)

```bash
RESOURCES_DIR="$SCRIPT_DIR/pre-push-resources"  # Path to resources directory
MIN_CHANGES=15                                   # Minimum added lines to trigger review
PROMPT_TEMPLATE_FILE="$RESOURCES_DIR/prompt_templates/review_prompt_template.txt"
```

### File Filtering Logic (pre-push:134-146)

Files are excluded from review if path contains:
- `enum`
- `constant`
- `.yml` extension

### Issue Detection Logic (pre-push:263-277)

The hook blocks push if any of these sections don't contain "_No se encontraron problemas_":
- `## Posibles Bugs`
- `## Posibles problemas de Diseño / SOLID`
- `## Posibles problemas de Clean Code`
- `## Violaciones de lineamientos REST`
- `## Violaciones de lineamientos de Swagger`

## Important Implementation Details

### API Communication
- Uses `curl` for HTTP POST to Gemini API
- Escapes JSON strings properly (handles backslashes, quotes, newlines)
- Creates temporary files for payload and response in `/tmp/`
- Extracts response using `jq` if available, falls back to raw output

### Git Diff Generation
- Smart branch comparison based on branch naming patterns
- Falls back to parent commit if target branch doesn't exist
- For initial commits, uses empty tree object as parent

### Error Handling
- Validates API key presence before proceeding
- Checks for empty diffs (skips review)
- Validates minimum change threshold
- Provides clear error messages for missing dependencies or configuration

## Modifying Review Criteria

To change what gets reviewed or how:

1. **Adjust minimum changes**: Edit `MIN_CHANGES` variable in pre-push:30
2. **Modify file filters**: Edit `is_relevant_file()` function in pre-push (around line 224)
3. **Change review focus**: Edit `pre-push-resources/prompt_templates/review_prompt_template.txt`
4. **Customize PR format**: Edit `pre-push-resources/prompt_templates/pr_details_prompt_template.txt`
5. **Add/remove issue sections**: Update `has_no_issues()` function in pre-push (around line 370)
6. **Change API configuration**: Edit `pre-push-resources/gemini.ini`

## Working with Gemini API

The scripts use Gemini 2.5 Pro model by default. The API request structure:
```json
{
  "contents": [
    {
      "parts": [
        {
          "text": "<escaped prompt with git diff>"
        }
      ]
    }
  ]
}
```

Response is parsed from: `.candidates[0].content.parts[0].text`
