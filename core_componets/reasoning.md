# Reasoning

> Learn how to enable and use agent reasoning to improve task execution.

## Overview

Agent reasoning is a feature that allows agents to reflect on a task and create a plan before execution. This helps agents approach tasks more methodically and ensures they're ready to perform the assigned work.

## Usage

To enable reasoning for an agent, simply set `reasoning=True` when creating the agent:

```python
from crewai import Agent

agent = Agent(
    role="Data Analyst",
    goal="Analyze complex datasets and provide insights",
    backstory="You are an experienced data analyst with expertise in finding patterns in complex data.",
    reasoning=True,  # Enable reasoning
    max_reasoning_attempts=3  # Optional: Set a maximum number of reasoning attempts
)
```

## How It Works

When reasoning is enabled, before executing a task, the agent will:

1. Reflect on the task and create a detailed plan
2. Evaluate whether it's ready to execute the task
3. Refine the plan as necessary until it's ready or max\_reasoning\_attempts is reached
4. Inject the reasoning plan into the task description before execution

This process helps the agent break down complex tasks into manageable steps and identify potential challenges before starting.

## Configuration Options

<ParamField body="reasoning" type="bool" default="False">
  Enable or disable reasoning
</ParamField>

<ParamField body="max_reasoning_attempts" type="int" default="None">
  Maximum number of attempts to refine the plan before proceeding with execution. If None (default), the agent will continue refining until it's ready.
</ParamField>

## Example

Here's a complete example:

```python
from crewai import Agent, Task, Crew

# Create an agent with reasoning enabled
analyst = Agent(
    role="Data Analyst",
    goal="Analyze data and provide insights",
    backstory="You are an expert data analyst.",
    reasoning=True,
    max_reasoning_attempts=3  # Optional: Set a limit on reasoning attempts
)

# Create a task
analysis_task = Task(
    description="Analyze the provided sales data and identify key trends.",
    expected_output="A report highlighting the top 3 sales trends.",
    agent=analyst
)

# Create a crew and run the task
crew = Crew(agents=[analyst], tasks=[analysis_task])
result = crew.kickoff()

print(result)
```

## Error Handling

The reasoning process is designed to be robust, with error handling built in. If an error occurs during reasoning, the agent will proceed with executing the task without the reasoning plan. This ensures that tasks can still be executed even if the reasoning process fails.

Here's how to handle potential errors in your code:

```python
from crewai import Agent, Task
import logging

# Set up logging to capture any reasoning errors
logging.basicConfig(level=logging.INFO)

# Create an agent with reasoning enabled
agent = Agent(
    role="Data Analyst",
    goal="Analyze data and provide insights",
    reasoning=True,
    max_reasoning_attempts=3
)

# Create a task
task = Task(
    description="Analyze the provided sales data and identify key trends.",
    expected_output="A report highlighting the top 3 sales trends.",
    agent=agent
)

# Execute the task
# If an error occurs during reasoning, it will be logged and execution will continue
result = agent.execute_task(task)
```

## Example Reasoning Output

Here's an example of what a reasoning plan might look like for a data analysis task:

```
Task: Analyze the provided sales data and identify key trends.

Reasoning Plan:
I'll analyze the sales data to identify the top 3 trends.

1. Understanding of the task:
   I need to analyze sales data to identify key trends that would be valuable for business decision-making.

2. Key steps I'll take:
   - First, I'll examine the data structure to understand what fields are available
   - Then I'll perform exploratory data analysis to identify patterns
   - Next, I'll analyze sales by time periods to identify temporal trends
   - I'll also analyze sales by product categories and customer segments
   - Finally, I'll identify the top 3 most significant trends

3. Approach to challenges:
   - If the data has missing values, I'll decide whether to fill or filter them
   - If the data has outliers, I'll investigate whether they're valid data points or errors
   - If trends aren't immediately obvious, I'll apply statistical methods to uncover patterns

4. Use of available tools:
   - I'll use data analysis tools to explore and visualize the data
   - I'll use statistical tools to identify significant patterns
   - I'll use knowledge retrieval to access relevant information about sales analysis

5. Expected outcome:
   A concise report highlighting the top 3 sales trends with supporting evidence from the data.

READY: I am ready to execute the task.
```

This reasoning plan helps the agent organize its approach to the task, consider potential challenges, and ensure it delivers the expected output.
