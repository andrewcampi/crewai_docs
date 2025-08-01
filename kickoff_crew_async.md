# Kickoff Crew Asynchronously

> Kickoff a Crew Asynchronously

## Introduction

CrewAI provides the ability to kickoff a crew asynchronously, allowing you to start the crew execution in a non-blocking manner.
This feature is particularly useful when you want to run multiple crews concurrently or when you need to perform other tasks while the crew is executing.

## Asynchronous Crew Execution

To kickoff a crew asynchronously, use the `kickoff_async()` method. This method initiates the crew execution in a separate thread, allowing the main thread to continue executing other tasks.

### Method Signature

```python Code
def kickoff_async(self, inputs: dict) -> CrewOutput:
```

### Parameters

* `inputs` (dict): A dictionary containing the input data required for the tasks.

### Returns

* `CrewOutput`: An object representing the result of the crew execution.

## Potential Use Cases

* **Parallel Content Generation**: Kickoff multiple independent crews asynchronously, each responsible for generating content on different topics. For example, one crew might research and draft an article on AI trends, while another crew generates social media posts about a new product launch. Each crew operates independently, allowing content production to scale efficiently.

* **Concurrent Market Research Tasks**: Launch multiple crews asynchronously to conduct market research in parallel. One crew might analyze industry trends, while another examines competitor strategies, and yet another evaluates consumer sentiment. Each crew independently completes its task, enabling faster and more comprehensive insights.

* **Independent Travel Planning Modules**: Execute separate crews to independently plan different aspects of a trip. One crew might handle flight options, another handles accommodation, and a third plans activities. Each crew works asynchronously, allowing various components of the trip to be planned simultaneously and independently for faster results.

## Example: Single Asynchronous Crew Execution

Here's an example of how to kickoff a crew asynchronously using asyncio and awaiting the result:

```python Code
import asyncio
from crewai import Crew, Agent, Task

# Create an agent with code execution enabled
coding_agent = Agent(
    role="Python Data Analyst",
    goal="Analyze data and provide insights using Python",
    backstory="You are an experienced data analyst with strong Python skills.",
    allow_code_execution=True
)

# Create a task that requires code execution
data_analysis_task = Task(
    description="Analyze the given dataset and calculate the average age of participants. Ages: {ages}",
    agent=coding_agent,
    expected_output="The average age of the participants."
)

# Create a crew and add the task
analysis_crew = Crew(
    agents=[coding_agent],
    tasks=[data_analysis_task]
)

# Async function to kickoff the crew asynchronously
async def async_crew_execution():
    result = await analysis_crew.kickoff_async(inputs={"ages": [25, 30, 35, 40, 45]})
    print("Crew Result:", result)

# Run the async function
asyncio.run(async_crew_execution())
```

## Example: Multiple Asynchronous Crew Executions

In this example, we'll show how to kickoff multiple crews asynchronously and wait for all of them to complete using `asyncio.gather()`:

```python Code
import asyncio
from crewai import Crew, Agent, Task

# Create an agent with code execution enabled
coding_agent = Agent(
    role="Python Data Analyst",
    goal="Analyze data and provide insights using Python",
    backstory="You are an experienced data analyst with strong Python skills.",
    allow_code_execution=True
)

# Create tasks that require code execution
task_1 = Task(
    description="Analyze the first dataset and calculate the average age of participants. Ages: {ages}",
    agent=coding_agent,
    expected_output="The average age of the participants."
)

task_2 = Task(
    description="Analyze the second dataset and calculate the average age of participants. Ages: {ages}",
    agent=coding_agent,
    expected_output="The average age of the participants."
)

# Create two crews and add tasks
crew_1 = Crew(agents=[coding_agent], tasks=[task_1])
crew_2 = Crew(agents=[coding_agent], tasks=[task_2])

# Async function to kickoff multiple crews asynchronously and wait for all to finish
async def async_multiple_crews():
    # Create coroutines for concurrent execution
    result_1 = crew_1.kickoff_async(inputs={"ages": [25, 30, 35, 40, 45]})
    result_2 = crew_2.kickoff_async(inputs={"ages": [20, 22, 24, 28, 30]})

    # Wait for both crews to finish
    results = await asyncio.gather(result_1, result_2)

    for i, result in enumerate(results, 1):
        print(f"Crew {i} Result:", result)

# Run the async function
asyncio.run(async_multiple_crews())
```
