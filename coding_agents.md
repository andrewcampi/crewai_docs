# Coding Agents

> Learn how to enable your CrewAI Agents to write and execute code, and explore advanced features for enhanced functionality.

## Introduction

CrewAI Agents now have the powerful ability to write and execute code, significantly enhancing their problem-solving capabilities. This feature is particularly useful for tasks that require computational or programmatic solutions.

## Enabling Code Execution

To enable code execution for an agent, set the `allow_code_execution` parameter to `True` when creating the agent.

Here's an example:

```python Code
from crewai import Agent

coding_agent = Agent(
    role="Senior Python Developer",
    goal="Craft well-designed and thought-out code",
    backstory="You are a senior Python developer with extensive experience in software architecture and best practices.",
    allow_code_execution=True
)
```

<Note>
  Note that `allow_code_execution` parameter defaults to `False`.
</Note>

## Important Considerations

1. **Model Selection**: It is strongly recommended to use more capable models like Claude 3.5 Sonnet and GPT-4 when enabling code execution.
   These models have a better understanding of programming concepts and are more likely to generate correct and efficient code.

2. **Error Handling**: The code execution feature includes error handling. If executed code raises an exception, the agent will receive the error message and can attempt to correct the code or
   provide alternative solutions. The `max_retry_limit` parameter, which defaults to 2, controls the maximum number of retries for a task.

3. **Dependencies**: To use the code execution feature, you need to install the `crewai_tools` package. If not installed, the agent will log an info message:
   "Coding tools not available. Install crewai\_tools."

## Code Execution Process

When an agent with code execution enabled encounters a task requiring programming:

<Steps>
  <Step title="Task Analysis">
    The agent analyzes the task and determines that code execution is necessary.
  </Step>

  <Step title="Code Formulation">
    It formulates the Python code needed to solve the problem.
  </Step>

  <Step title="Code Execution">
    The code is sent to the internal code execution tool (`CodeInterpreterTool`).
  </Step>

  <Step title="Result Interpretation">
    The agent interprets the result and incorporates it into its response or uses it for further problem-solving.
  </Step>
</Steps>

## Example Usage

Here's a detailed example of creating an agent with code execution capabilities and using it in a task:

```python Code
from crewai import Agent, Task, Crew

# Create an agent with code execution enabled
coding_agent = Agent(
    role="Python Data Analyst",
    goal="Analyze data and provide insights using Python",
    backstory="You are an experienced data analyst with strong Python skills.",
    allow_code_execution=True
)

# Create a task that requires code execution
data_analysis_task = Task(
    description="Analyze the given dataset and calculate the average age of participants.",
    agent=coding_agent
)

# Create a crew and add the task
analysis_crew = Crew(
    agents=[coding_agent],
    tasks=[data_analysis_task]
)

# Execute the crew
result = analysis_crew.kickoff()

print(result)
```

In this example, the `coding_agent` can write and execute Python code to perform data analysis tasks.
