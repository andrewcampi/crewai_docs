# CLI

> Learn how to use the CrewAI CLI to interact with CrewAI.

<Warning>Since release 0.140.0, CrewAI Enterprise started a process of migrating their login provider. As such, the authentication flow via CLI was updated. Users that use Google to login, or that created their account after July 3rd, 2025 will be unable to log in with older versions of the `crewai` library.</Warning>

## Overview

The CrewAI CLI provides a set of commands to interact with CrewAI, allowing you to create, train, run, and manage crews & flows.

## Installation

To use the CrewAI CLI, make sure you have CrewAI installed:

```shell Terminal
pip install crewai
```

## Basic Usage

The basic structure of a CrewAI CLI command is:

```shell Terminal
crewai [COMMAND] [OPTIONS] [ARGUMENTS]
```

## Available Commands

### 1. Create

Create a new crew or flow.

```shell Terminal
crewai create [OPTIONS] TYPE NAME
```

* `TYPE`: Choose between "crew" or "flow"
* `NAME`: Name of the crew or flow

Example:

```shell Terminal
crewai create crew my_new_crew
crewai create flow my_new_flow
```

### 2. Version

Show the installed version of CrewAI.

```shell Terminal
crewai version [OPTIONS]
```

* `--tools`: (Optional) Show the installed version of CrewAI tools

Example:

```shell Terminal
crewai version
crewai version --tools
```

### 3. Train

Train the crew for a specified number of iterations.

```shell Terminal
crewai train [OPTIONS]
```

* `-n, --n_iterations INTEGER`: Number of iterations to train the crew (default: 5)
* `-f, --filename TEXT`: Path to a custom file for training (default: "trained\_agents\_data.pkl")

Example:

```shell Terminal
crewai train -n 10 -f my_training_data.pkl
```

### 4. Replay

Replay the crew execution from a specific task.

```shell Terminal
crewai replay [OPTIONS]
```

* `-t, --task_id TEXT`: Replay the crew from this task ID, including all subsequent tasks

Example:

```shell Terminal    
crewai replay -t task_123456
```

### 5. Log-tasks-outputs

Retrieve your latest crew\.kickoff() task outputs.

```shell Terminal
crewai log-tasks-outputs
```

### 6. Reset-memories

Reset the crew memories (long, short, entity, latest\_crew\_kickoff\_outputs).

```shell Terminal
crewai reset-memories [OPTIONS]
```

* `-l, --long`: Reset LONG TERM memory
* `-s, --short`: Reset SHORT TERM memory
* `-e, --entities`: Reset ENTITIES memory
* `-k, --kickoff-outputs`: Reset LATEST KICKOFF TASK OUTPUTS
* `-kn, --knowledge`: Reset KNOWLEDGE storage
* `-akn, --agent-knowledge`: Reset AGENT KNOWLEDGE storage
* `-a, --all`: Reset ALL memories

Example:

```shell Terminal
crewai reset-memories --long --short
crewai reset-memories --all
```

### 7. Test

Test the crew and evaluate the results.

```shell Terminal
crewai test [OPTIONS]
```

* `-n, --n_iterations INTEGER`: Number of iterations to test the crew (default: 3)
* `-m, --model TEXT`: LLM Model to run the tests on the Crew (default: "gpt-4o-mini")

Example:

```shell Terminal    
crewai test -n 5 -m gpt-3.5-turbo
```

### 8. Run

Run the crew or flow.

```shell Terminal
crewai run
```

<Note>
  Starting from version 0.103.0, the `crewai run` command can be used to run both standard crews and flows. For flows, it automatically detects the type from pyproject.toml and runs the appropriate command. This is now the recommended way to run both crews and flows.
</Note>

<Note>
  Make sure to run these commands from the directory where your CrewAI project is set up.
  Some commands may require additional configuration or setup within your project structure.
</Note>

### 9. Chat

Starting in version `0.98.0`, when you run the `crewai chat` command, you start an interactive session with your crew. The AI assistant will guide you by asking for necessary inputs to execute the crew. Once all inputs are provided, the crew will execute its tasks.

After receiving the results, you can continue interacting with the assistant for further instructions or questions.

```shell Terminal
crewai chat
```

<Note>
  Ensure you execute these commands from your CrewAI project's root directory.
</Note>

<Note>
  IMPORTANT: Set the `chat_llm` property in your `crew.py` file to enable this command.

  ```python
  @crew
  def crew(self) -> Crew:
      return Crew(
          agents=self.agents,
          tasks=self.tasks,
          process=Process.sequential,
          verbose=True,
          chat_llm="gpt-4o",  # LLM for chat orchestration
      )
  ```
</Note>

### 10. Deploy

Deploy the crew or flow to [CrewAI Enterprise](https://app.crewai.com).

* **Authentication**: You need to be authenticated to deploy to CrewAI Enterprise.
  You can login or create an account with:
  ```shell Terminal
  crewai login
  ```

* **Create a deployment**: Once you are authenticated, you can create a deployment for your crew or flow from the root of your localproject.
  ```shell Terminal
  crewai deploy create
  ```
  * Reads your local project configuration.
  * Prompts you to confirm the environment variables (like `OPENAI_API_KEY`, `SERPER_API_KEY`) found locally. These will be securely stored with the deployment on the Enterprise platform. Ensure your sensitive keys are correctly configured locally (e.g., in a `.env` file) before running this.

### 11. Organization Management

Manage your CrewAI Enterprise organizations.

```shell Terminal
crewai org [COMMAND] [OPTIONS]
```

#### Commands:

* `list`: List all organizations you belong to

```shell Terminal
crewai org list
```

* `current`: Display your currently active organization

```shell Terminal
crewai org current
```

* `switch`: Switch to a specific organization

```shell Terminal
crewai org switch <organization_id>
```

<Note>
  You must be authenticated to CrewAI Enterprise to use these organization management commands.
</Note>

* **Create a deployment** (continued):
  * Links the deployment to the corresponding remote GitHub repository (it usually detects this automatically).

* **Deploy the Crew**: Once you are authenticated, you can deploy your crew or flow to CrewAI Enterprise.
  ```shell Terminal
  crewai deploy push
  ```
  * Initiates the deployment process on the CrewAI Enterprise platform.
  * Upon successful initiation, it will output the Deployment created successfully! message along with the Deployment Name and a unique Deployment ID (UUID).

* **Deployment Status**: You can check the status of your deployment with:
  ```shell Terminal
  crewai deploy status
  ```
  This fetches the latest deployment status of your most recent deployment attempt (e.g., `Building Images for Crew`, `Deploy Enqueued`, `Online`).

* **Deployment Logs**: You can check the logs of your deployment with:
  ```shell Terminal
  crewai deploy logs
  ```
  This streams the deployment logs to your terminal.

* **List deployments**: You can list all your deployments with:
  ```shell Terminal
  crewai deploy list
  ```
  This lists all your deployments.

* **Delete a deployment**: You can delete a deployment with:
  ```shell Terminal
  crewai deploy remove
  ```
  This deletes the deployment from the CrewAI Enterprise platform.

* **Help Command**: You can get help with the CLI with:
  ```shell Terminal
  crewai deploy --help
  ```
  This shows the help message for the CrewAI Deploy CLI.

Watch this video tutorial for a step-by-step demonstration of deploying your crew to [CrewAI Enterprise](http://app.crewai.com) using the CLI.

<iframe width="100%" height="400" src="https://www.youtube.com/embed/3EqSV-CYDZA" title="CrewAI Deployment Guide" frameborder="0" style={{ borderRadius: '10px' }} allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen />

### 11. API Keys

When running `crewai create crew` command, the CLI will show you a list of available LLM providers to choose from, followed by model selection for your chosen provider.

Once you've selected an LLM provider and model, you will be prompted for API keys.

#### Available LLM Providers

Here's a list of the most popular LLM providers suggested by the CLI:

* OpenAI
* Groq
* Anthropic
* Google Gemini
* SambaNova

When you select a provider, the CLI will then show you available models for that provider and prompt you to enter your API key.

#### Other Options

If you select "other", you will be able to select from a list of LiteLLM supported providers.

When you select a provider, the CLI will prompt you to enter the Key name and the API key.

See the following link for each provider's key name:

* [LiteLLM Providers](https://docs.litellm.ai/docs/providers)
