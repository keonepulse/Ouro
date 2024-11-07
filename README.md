# Ouro

Ouro is an experimental framework for a self-building autonomous agent. Its guiding idea is that the optimal way to build a general autonomous agent is to build the simplest thing that can build itself — an agent that writes, stores, and reuses its own functions.

> **Caution:** This is an experimental framework meant to share ideas, spark discussion, and let experienced developers explore the concept of self-building agents. It is **not** meant for production use.

At its core is a function framework (**functionz**) for storing, managing, and executing functions from a database. It offers a graph-based structure for tracking imports, dependent functions, and authentication secrets, with automatic loading and comprehensive logging. It also ships with a web dashboard for managing functions, running updates, and viewing logs.

> **Note:** The importable Python module is named `babyagi` for compatibility with existing code and examples.

## Table of Contents

- [Quick Start](#quick-start)
- [Basic Usage](#basic-usage)
- [Function Metadata](#function-metadata)
- [Function Loading](#function-loading)
- [Key Dependencies](#key-dependencies)
- [Execution Environment](#execution-environment)
  - [Logging](#logging)
- [Dashboard](#dashboard)
- [Pre-loaded Functions](#pre-loaded-functions)
- [Running a Self-Building Agent](#running-a-self-building-agent)
- [Project Structure](#project-structure)
- [Contributing](#contributing)
- [License](#license)

## Quick Start

To quickly check out the dashboard and see how it works:

1. **Install from the project root:**

    ```bash
    pip install .
    ```

2. **Import the framework and load the dashboard:**

    ```python
    import babyagi

    if __name__ == "__main__":
        app = babyagi.create_app('/dashboard')
        app.run(host='0.0.0.0', port=8080)
    ```

3. **Navigate to the dashboard:**

    Open your browser and go to `http://localhost:8080/dashboard`.

## Basic Usage

Start by importing the framework and registering your functions. Here's how to register two functions, where one depends on the other:

```python
import babyagi

# Register a simple function
@babyagi.register_function()
def world():
    return "world"

# Register a function that depends on 'world'
@babyagi.register_function(dependencies=["world"])
def hello_world():
    x = world()
    return f"Hello {x}!"

# Execute the function
print(babyagi.hello_world())  # Output: Hello world!

if __name__ == "__main__":
    app = babyagi.create_app('/dashboard')
    app.run(host='0.0.0.0', port=8080)
```

## Function Metadata

Functions can be registered with metadata to enhance their capabilities and manage their relationships. Here's a comprehensive example showing all fields:

```python
import babyagi

@babyagi.register_function(
    imports=["math"],
    dependencies=["circle_area"],
    key_dependencies=["openai_api_key"],
    metadata={
        "description": "Calculates the volume of a cylinder using the circle_area function."
    }
)
def cylinder_volume(radius, height):
    import math
    area = circle_area(radius)
    return area * height
```

**Available Metadata Fields:**

- `imports` — list of external libraries the function depends on.
- `dependencies` — list of other functions this function depends on.
- `key_dependencies` — list of secret keys required by the function.
- `metadata["description"]` — a description of what the function does.

## Function Loading

In addition to `register_function`, you can use `load_functions` to load plugins or packs of functions. The framework comes with built-in function packs, or you can load your own by pointing to a file path.

Available function packs live in `babyagi/functionz/packs`.

**Loading Custom Function Packs:**

```python
import babyagi

# Load your custom function pack
babyagi.load_functions("path/to/your/custom_functions.py")
```

This approach keeps function building and management organized by grouping related functions into packs.

## Key Dependencies

You can store secret keys (`key_dependencies`) directly from code or manage them via the dashboard.

**Storing Key Dependencies from Code:**

```python
import babyagi

# Add a secret key
babyagi.add_key_wrapper('openai_api_key', 'your_openai_api_key')
```

**Adding Key Dependencies via Dashboard:**

Navigate to the dashboard and use the **add_key_wrapper** feature to securely add your secret keys.

## Execution Environment

The framework automatically loads essential function packs and manages their dependencies, ensuring a seamless execution environment. It also logs all activity, including the relationships between functions, to provide comprehensive tracking of executions and dependencies.

### Logging

A comprehensive logging system tracks all function executions and their interactions. Every function call — including inputs, outputs, execution time, and errors — is recorded for monitoring and debugging.

**Key Logging Features:**

- **Execution Tracking** — logs when a function starts and finishes, including its name, arguments, keyword arguments, and execution time.
- **Error Logging** — captures errors during execution with detailed messages for troubleshooting.
- **Dependency Management** — automatically resolves and logs dependencies, ensuring required functions and libraries are loaded before execution.
- **Trigger Logging** — records which functions were triggered by others and their execution outcomes.
- **Comprehensive Records** — maintains a history of all executions, enabling review of past activity, function relationships, and performance.

**How Triggers Work:**

Triggers automatically execute certain functions in response to events within the system. For example, when a function is added or updated, a trigger can generate a description for it. Triggers increase the framework's autonomy by enabling automated workflows — but manage them carefully to avoid unintended recursive executions or conflicts between dependent functions.

## Dashboard

The dashboard offers a user-friendly interface for managing functions, monitoring executions, and handling configuration:

- **Function Management** — register, deregister, and update functions directly from the dashboard.
- **Dependency Visualization** — view and manage relationships between functions.
- **Secret Key Management** — add and manage secret keys securely.
- **Logging and Monitoring** — access comprehensive execution logs, including inputs, outputs, and timings.
- **Trigger Management** — set up triggers to automate function execution based on events or conditions.

After running your application, navigate to `http://localhost:8080/dashboard`.

## Pre-loaded Functions

Two function packs are pre-loaded:

1. **Default Functions (`packs/default_functions.py`):**
   - **Function Execution** — run, add, update, or retrieve functions and versions.
   - **Key Management** — add and retrieve secret keys.
   - **Triggers** — add triggers that execute functions based on others.
   - **Logs** — retrieve logs with optional filters.

2. **AI Functions (`packs/ai_generator.py`):**
   - **AI Descriptions & Embeddings** — auto-generate descriptions and embeddings for functions.
   - **Function Selection** — find or choose similar functions based on prompts.

## Running a Self-Building Agent

Two experimental self-building agents demonstrate how the framework helps a coding agent leverage existing functions to write new ones.

### 1. `process_user_input` in the `code_writing_functions` pack

This function first decides whether to use an existing function or generate new ones. If new functions are needed, it breaks them into smaller reusable components and combines them into a final function.

```python
import os
import babyagi

babyagi.add_key_wrapper('openai_api_key', os.environ['OPENAI_API_KEY'])
babyagi.load_functions("drafts/code_writing_functions")

babyagi.process_user_input("Grab today's score from ESPN and email it to test@example.com")
```

When you run this, you will see functions being generated in the shell, and the new functions appear in the dashboard once completed.

### 2. `self_build` in the `self_build` pack

This function takes a user description and generates X distinct tasks a user might ask an AI assistant, then routes each task through `process_user_input`, creating new functions when no existing ones suffice.

```python
import os
import babyagi

babyagi.add_key_wrapper('openai_api_key', os.environ['OPENAI_API_KEY'])
babyagi.load_functions("drafts/code_writing_functions")
babyagi.load_functions("drafts/self_build")

babyagi.self_build("A sales person at an enterprise SaaS company.", 3)
```

This generates three distinct tasks a salesperson might ask an AI assistant and creates functions to handle them. The generated functions are stored in the dashboard, but note that the generated code is minimal and may need improvement.

> **Warning:** These draft features are experimental concepts and may not function as intended. They require significant improvements and should be used with caution.

## Project Structure

```
.
├── babyagi/                  # Importable Python package
│   ├── api/                  # REST API endpoints
│   ├── dashboard/            # Web dashboard (templates, static assets)
│   └── functionz/            # Core function framework
│       ├── core/             # Registration, execution, framework logic
│       ├── db/               # Database models and access
│       └── packs/            # Built-in, draft, and plugin function packs
├── examples/                 # Usage examples (quickstart, triggers, custom routes, ...)
├── main.py                   # Example entry point with dashboard
├── requirements.txt          # Python dependencies
├── setup.py                  # Package setup
└── CODE_READINESS_ANALYSIS.md  # Honest assessment of production readiness
```

## Contributing

Contributions are welcome. Please keep in mind that this is an experimental project, so expect discussion before larger changes are merged. Reviewing the analysis in `CODE_READINESS_ANALYSIS.md` is a good starting point for finding areas that need work — security hardening and test coverage are the most valuable contributions.

## License

Ouro is released under the MIT License.
