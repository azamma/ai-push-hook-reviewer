# AI-Powered Git Hooks for Code Review and PR Details

This project provides a set of scripts that leverage the Gemini API to automate parts of the development workflow, including code reviews and pull request detail generation.

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
  - `gemini.ini`: Configuration file for Gemini API key and URL
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

4. **Configure your Gemini API key:**

   - Get your API key from: https://aistudio.google.com/app/apikey
   - Edit `~/.git-hooks/pre-push-resources/gemini.ini`
   - Replace `your-gemini-api-key-here` with your actual API key:

   ```ini
   [gemini]
   api_key = YOUR-API-KEY-HERE
   api_url = https://generativelanguage.googleapis.com/v1/models/gemini-2.5-pro:generateContent
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

If you're on **Windows**, all `git push` commands must be executed from **Git Bash**, not from CMD or PowerShell. Environment variables like `SKIP` will not work correctly in CMD/PowerShell.

### Normal Usage

Once configured, the hook runs automatically every time you execute `git push` from Git Bash:

```bash
git push
```

The hook will:
1. Review your code changes for issues
2. Block the push if critical problems are found
3. Generate a `PR_DETAILS_<branch>_<timestamp>.md` file with PR documentation

### Bypassing the Hook

You can control which parts of the AI processing to skip using the `SKIP` environment variable:

```bash
# Skip only the AI review (PR details will still be generated)
SKIP=10 git push

# Skip only the PR details generation (AI review will still run)
SKIP=01 git push

# Skip both AI review and PR details generation
SKIP=11 git push
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

### "Failed to call Gemini API"
- Check your internet connection
- Verify your `api_key` in `gemini.ini` is correct
- Ensure the API URL is correct

### "gemini.ini not found"
- For global setup: Check that `~/.git-hooks/pre-push-resources/gemini.ini` exists
- For per-project setup: Check that `.git/hooks/pre-push-resources/gemini.ini` exists

## Customization

You can customize the AI behavior by editing the prompt templates:

- **Code Review:** Edit `~/.git-hooks/pre-push-resources/prompt_templates/review_prompt_template.txt`
- **PR Details:** Edit `~/.git-hooks/pre-push-resources/prompt_templates/pr_details_prompt_template.txt`

## Contributing

Feel free to open issues or submit pull requests to improve this tool!
