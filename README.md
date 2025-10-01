# AI-Powered Git Hooks for Code Review and PR Details

This project provides a set of scripts that leverage the Gemini API to automate parts of the development workflow, including code reviews and pull request detail generation.

## File Structure

- `pre-push`: A git hook script that automatically reviews code changes before they are pushed. It will also trigger the `pr-details` script.
- `pr-details`: A script that generates a detailed description of the changes in a push, suitable for a pull request body. This script is called automatically by the `pre-push` hook.
- `gemini.ini`: The configuration file for storing your Gemini API key and API URL.
- `prompt_templates/`: A directory containing the prompt templates for the different scripts.

All these files and directories should be placed inside the `.git/hooks` directory of your target repository.

## Configuration

Both scripts use the `gemini.ini` file for configuration. This file should be placed in your `.git/hooks` directory.

Create the `gemini.ini` file with the following content:

```ini
[gemini]
api_key = your-gemini-api-key-here
api_url = https://generativelanguage.googleapis.com/v1/models/gemini-2.5-pro:generateContent
```

Replace `your-gemini-api-key-here` with your actual Gemini API key.

## Scripts

### `pre-push` Hook

The `pre-push` script is a git hook that automatically reviews your code for potential issues before you push it. After the review, it will always execute the `pr-details` script to generate the PR details. If the review finds any critical problems, it will still block the push.

#### Setup

1.  **Copy all the scripts and configuration files to your git hooks directory:**
    ```bash
    cp pre-push .git/hooks/
    cp pr-details .git/hooks/
    cp gemini.ini .git/hooks/
    cp -r prompt_templates .git/hooks/
    ```
2.  **Make the scripts executable:**
    ```bash
    chmod +x .git/hooks/pre-push
    chmod +x .git/hooks/pr-details
    ```

Now, the hook will run automatically every time you `git push`.

#### Bypassing the hook

You can control which parts of the AI processing to skip using the `SKIP_AI` environment variable:

```bash
# Skip only the AI review (PR details will still be generated)
SKIP_AI=10 git push

# Skip only the PR details generation (AI review will still run)
SKIP_AI=01 git push

# Skip both AI review and PR details generation
SKIP_AI=11 git push
```

**Legacy flag**: The `SKIP_AI_REVIEW` variable is still supported for backward compatibility:
```bash
SKIP_AI_REVIEW=1 git push  # Same as SKIP_AI=10
```

### `pr-details` Script

The `pr-details` script generates a detailed summary of the changes you have pushed. It is designed to be called automatically by the `pre-push` hook.

It will create a new Markdown file in the root of the repository with a name like `PR_DETAILS_<branch-name>_<timestamp>.md`.

## Dependencies

- `bash`
- `curl`
- `git`
- `jq` (optional, for better JSON parsing)

### Installing `jq`

`jq` is used for more reliable parsing of the JSON response from the Gemini API.

-   **Ubuntu/Debian:**
    ```bash
    sudo apt-get install jq
    ```
-   **CentOS/RHEL:**
    ```bash
    sudo yum install jq
    ```
-   **macOS:**
    ```bash
    brew install jq
    ```
-   **Windows (using Chocolatey):**
    ```bash
    choco install jq
    ```

## Troubleshooting

- **"curl command not found"**: Install `curl` on your system.
- **"Failed to call Gemini API"**: Check your internet connection and ensure your `api_key` in `gemini.ini` is correct.
- **Hook not running**: Make sure the `pre-push` script is in the `.git/hooks` directory and is executable.
- **"gemini.ini not found"**: Make sure the `gemini.ini` file is in your `.git/hooks` directory.
