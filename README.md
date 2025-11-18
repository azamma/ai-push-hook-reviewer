# AI-Powered Git Hooks for Code Review and PR Details

This project provides a set of scripts that leverage AI APIs (Gemini, Deepseek, and more) to automate parts of the development workflow, including code reviews and pull request detail generation.

**Multi-Model Support**: Switch between different AI models at runtime using environment variables!

## What Does It Do?

✅ **Automatic code review** before each push
✅ **Blocks pushes** with critical issues detected
✅ **Generates PR documentation** in markdown automatically
✅ Validates REST guidelines, Swagger, SOLID and Clean Code
✅ Detects potential bugs, design issues and edge cases

## File Structure

- `pre-push`: The main git hook script that automatically reviews code changes before they are pushed
- `pre-push-resources/`: Directory containing all supporting files:
  - `pr-details`: Script that generates detailed PR descriptions
  - `ai-models.yml`: **NEW** Configuration file defining all available AI models
  - `gemini.ini`: Configuration file for Gemini API key and URL
  - `deepseek.ini`: Configuration file for Deepseek API key and URL
  - `prompt_templates/`: Directory with prompt templates for both scripts
    - `review_prompt_template.txt`: Template for code review
    - `pr_details_prompt_template.txt`: Template for PR details generation

## Setup (Recommended: Global Configuration)

### Prerequisites

**Install jq** (optional but recommended for better JSON parsing):

- **Windows (Chocolatey):**

  First install Chocolatey. Open **PowerShell as Administrator** and run:
  ```powershell
  Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
  ```

  Then install jq:
  ```powershell
  choco install jq -y
  ```

- **Ubuntu/Debian:**
  ```bash
  sudo apt-get install jq -y
  ```

- **macOS:**
  ```bash
  brew install jq
  ```

### Global Setup (Works for All Your Projects)

This is the **recommended approach** - configure once and it works for all your repositories.

1. **Configure global hooks directory:**

   Open Git Bash and run:
   ```bash
   mkdir -p ~/.git-hooks
   git config --global core.hooksPath ~/.git-hooks
   ```

2. **Clone this repository:**
   ```bash
   cd ~
   git clone https://github.com/azamma/ai-push-hook-reviewer.git
   ```

3. **Copy files to global hooks directory:**
   ```bash
   cp ~/ai-push-hook-reviewer/pre-push ~/.git-hooks/
   cp -r ~/ai-push-hook-reviewer/pre-push-resources ~/.git-hooks/
   chmod +x ~/.git-hooks/pre-push
   chmod +x ~/.git-hooks/pre-push-resources/pr-details
   ```

4. **Configure your API keys:**

   **For Gemini (Default Model):**
   - Get your API key from: https://aistudio.google.com/app/apikey
   - Edit `~/.git-hooks/pre-push-resources/gemini.ini`
   - Replace `your-gemini-api-key-here` with your actual API key:

   ```ini
   [gemini]
   api_key = YOUR-API-KEY-HERE
   api_url = https://generativelanguage.googleapis.com/v1/models/gemini-2.5-pro:generateContent
   ```

   **For Deepseek (Optional - Alternative Model):**
   - Get your API key from: https://platform.deepseek.com/api_keys
   - Edit `~/.git-hooks/pre-push-resources/deepseek.ini`
   - Replace `your-deepseek-api-key-here` with your actual API key:

   ```ini
   [deepseek]
   api_key = YOUR-API-KEY-HERE
   api_url = https://api.deepseek.com/v1/chat/completions
   model = deepseek-chat
   ```

Now the hook will run automatically in **all your Git repositories** every time you `git push`.

### Alternative: Per-Project Setup

If you prefer to configure it only for a specific repository:

1. **Copy files to your project's hooks directory:**
   ```bash
   cp pre-push .git/hooks/
   cp -r pre-push-resources .git/hooks/
   ```

2. **Make scripts executable:**
   ```bash
   chmod +x .git/hooks/pre-push
   chmod +x .git/hooks/pre-push-resources/pr-details
   ```

3. **Configure your API key:**
   ```bash
   # Edit the gemini.ini file with your API key
   nano .git/hooks/pre-push-resources/gemini.ini
   ```

## Usage

### ⚠️ IMPORTANT: Use Git Bash on Windows

If you're on **Windows**, all `git push` commands must be executed from **Git Bash**, not from CMD or PowerShell. Environment variables like `SKIP` and `MODEL` will not work correctly in CMD/PowerShell.

### Selecting AI Models

The system supports multiple AI models that can be selected at runtime using the `MODEL` environment variable:

| Model ID | Name | Config File | Default |
|----------|------|-------------|---------|
| 1 | Gemini (2.5-pro) | `gemini.ini` | ✅ Yes |
| 2 | Deepseek (Chat) | `deepseek.ini` | No |

**Usage Examples:**

```bash
# Use default Model 1 (Gemini)
git push

# Use Model 2 (Deepseek)
MODEL=2 git push

# Use Model 1 explicitly (same as default)
MODEL=1 git push
```

### Normal Usage

Once configured, the hook runs automatically every time you execute `git push` from Git Bash:

```bash
git push
```

The hook will:
1. Review your code changes for issues using the selected AI model
2. Block the push if critical problems are found
3. Generate a `PR_DETAILS_<branch>_<timestamp>.md` file with PR documentation

### Branch naming and diff base selection

The PR details script inspects the branch name to determine which base branch to use when building the diff:

- Branches containing `/ci/` (for example `azamma/ci/GROW-253`) are compared against `master`.
- Branches containing `/dev/` (for example `azamma/dev/GROW-253`) are compared against `development`.
- Branches containing `/release/` (for example `azamma/release/GROW-253`) are compared against `release`.
- Any other branch falls back to the default behaviour (diff against the push range, or against the tracked remote/parent commit when the script runs standalone).

If the resolved base branch is not available locally the script automatically falls back to the default diff logic, so you can keep working even when you only have the `origin/<branch>` references.

### Bypassing the Hook and Combining with Model Selection

You can control which parts of the AI processing to skip using the `SKIP` environment variable. You can also combine this with model selection:

```bash
# Skip only the AI review (PR details will still be generated)
SKIP=10 git push

# Skip only the PR details generation (AI review will still run)
SKIP=01 git push

# Skip both AI review and PR details generation
SKIP=11 git push

# Use Deepseek and skip PR details
MODEL=2 SKIP=01 git push

# Use Gemini and skip everything
MODEL=1 SKIP=11 git push
```

**Legacy flags**: For backward compatibility, `SKIP_AI` and `SKIP_AI_REVIEW` are still supported:
```bash
SKIP_AI=10 git push        # Deprecated, use SKIP=10
SKIP_AI_REVIEW=1 git push  # Deprecated, use SKIP=10
```

## What Gets Generated

### 1. Code Review (Console Output)

The tool analyzes your code and outputs a structured review with these sections:

- **Posibles Bugs** - Logic errors, edge cases, unhandled exceptions
- **Problemas de Diseño/SOLID** - Violations of SOLID principles, design issues
- **Problemas de Clean Code** - Readability, naming, duplication, magic numbers
- **Violaciones de lineamientos REST** - REST API guideline violations
- **Violaciones de lineamientos de Swagger** - Swagger/OpenAPI documentation issues

If critical issues are found in any section, the push will be **blocked**.

### 2. PR Details (Markdown File)

Automatically generates a professional `PR_DETAILS_<branch>_<timestamp>.md` file containing:

- 📊 PR metadata (assignee, ticket, dependencies, API Gateway, DB changes)
- 📝 Executive summary
- 🎯 Strategic entry point for code review
- 🏗️ Technical modifications with code snippets
- 📋 Index of modified classes
- 🧪 Test coverage
- 📈 Impact summary

## Requirements

- `bash` (included in Git Bash for Windows)
- `curl` (included in modern systems)
- `git`
- `jq` (optional but **highly recommended** for better JSON parsing - see installation above)

## Troubleshooting

### Hook not executing
```bash
git config --global core.hooksPath
# Should output: /c/Users/YourUser/.git-hooks (or equivalent)
```

If empty or incorrect, reconfigure:
```bash
git config --global core.hooksPath ~/.git-hooks
```

### SKIP variable not working (Windows)
- **Solution:** Use **Git Bash** instead of CMD or PowerShell

### Permission denied error
```bash
chmod +x ~/.git-hooks/pre-push
chmod +x ~/.git-hooks/pre-push-resources/pr-details
```

### "curl command not found"
- Install `curl` (should be available by default on modern systems)

### "Failed to call API"
- Check your internet connection
- Verify your `api_key` in the selected model config file is correct
- Ensure the API URL is correct in the config file

### "Model configuration not found" or "Model X not found in configuration"
- Ensure `ai-models.yml` exists in `~/.git-hooks/pre-push-resources/` (global setup)
- Verify the model ID exists in `ai-models.yml`
- Check that the referenced `.ini` files exist (gemini.ini, deepseek.ini, etc.)

### "gemini.ini not found" or "deepseek.ini not found"
- For global setup: Check that `~/.git-hooks/pre-push-resources/gemini.ini` and `deepseek.ini` exist
- For per-project setup: Check that `.git/hooks/pre-push-resources/gemini.ini` and `deepseek.ini` exist
- Copy the `.ini` template files from the repository if missing

## Customization

### Prompt Templates

You can customize the AI behavior by editing the prompt templates:

- **Code Review:** Edit `~/.git-hooks/pre-push-resources/prompt_templates/review_prompt_template.txt`
- **PR Details:** Edit `~/.git-hooks/pre-push-resources/prompt_templates/pr_details_prompt_template.txt`

### Adding New AI Models

To add support for a new AI model:

1. **Create a configuration file** (e.g., `claude.ini`):
   ```ini
   [claude]
   api_key = your-api-key-here
   api_url = https://api.anthropic.com/v1/messages
   model = claude-3-sonnet
   ```

2. **Update `ai-models.yml`** to add your model:
   ```yaml
   - id: 3
     name: Claude
     type: claude
     config_file: claude.ini
     description: "Anthropic Claude API"
     api_format: anthropic
     response_parser: ".content[0].text"
   ```

3. **Update the shell scripts** (`pre-push` and `pr-details`) to handle the new API format:
   - Add a new condition in the `call_ai_api()` function
   - Implement the appropriate API call (curl command with correct headers/auth)
   - Use the `RESPONSE_PARSER` variable from the YAML config

4. **Use your new model**:
   ```bash
   MODEL=3 git push
   ```

The system automatically loads the configuration from `ai-models.yml`, so new models are immediately available without modifying the core logic!

## Contributing

Feel free to open issues or submit pull requests to improve this tool!
