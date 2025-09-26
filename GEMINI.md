# Project Overview

This project provides a Git `pre-push` hook for automated code review using the Gemini API. It is designed to be used in a Spring Boot project, with a focus on maintaining code quality, adhering to design principles, and following specific REST and Swagger API guidelines.

The core of the project is a Bash script (`pre-push`) that gets triggered before a `git push`. This script intelligently detects the target branch, filters relevant files, and sends the code changes to the Gemini API for review. The review process is guided by a comprehensive prompt template (`prompt_template.txt`) that instructs the AI to act as a senior software engineer and check for various issues.

If the AI review identifies any critical problems (bugs, design flaws, etc.), the push is automatically blocked, forcing the developer to address the feedback. This helps ensure that only high-quality code that adheres to the project's standards is pushed to the repository.

## Building and Running

This project is not a traditional application that you build and run. Instead, it's a Git hook that you configure in your local development environment.

**To set up the `pre-push` hook:**

1.  **Copy the files:**
    Move the `pre-push` script and the `prompt_template.txt` file to the `.git/hooks/` directory of your Git repository.

2.  **Make the script executable:**
    ```bash
    chmod +x .git/hooks/pre-push
    ```

3.  **Configure your Gemini API Key:**
    Edit the `.git/hooks/pre-push` script and replace `"your-gemini-api-key-here"` with your actual Gemini API key.

    ```bash
    GEMINI_API_KEY="your-gemini-api-key-here"
    ```

**To use the hook:**

Once configured, the hook will run automatically every time you execute a `git push`.

**To temporarily bypass the review:**

If you need to push changes without triggering the AI review, you can use the `SKIP_AI_REVIEW` environment variable:

```bash
SKIP_AI_REVIEW=1 git push
```

## Development Conventions

The development conventions for this project are enforced by the AI code review process itself. The `prompt_template.txt` file defines the rules that the AI follows, which include:

*   **SOLID Principles:** The AI checks for violations of Single Responsibility, Open-Closed, Liskov Substitution, Interface Segregation, and Dependency Inversion principles.
*   **Clean Code:** The review process looks for issues related to readability, naming conventions, code duplication, magic numbers, and other clean code heuristics.
*   **REST API Guidelines:** The AI enforces specific guidelines for naming API endpoints, using HTTP methods correctly, structuring API composition, and versioning.
*   **Swagger/OpenAPI Documentation:** The review checks for adherence to the project's standards for documenting APIs using Swagger annotations.
*   **Error Handling:** The AI looks for potential bugs, unhandled exceptions, and edge cases.

By using this `pre-push` hook, developers can ensure that their code consistently meets the high-quality standards defined in the prompt template.
