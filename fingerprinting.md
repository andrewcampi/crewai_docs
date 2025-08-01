# Fingerprinting

> Learn how to use CrewAI's fingerprinting system to uniquely identify and track components throughout their lifecycle.

## Overview

Fingerprints in CrewAI provide a way to uniquely identify and track components throughout their lifecycle. Each `Agent`, `Crew`, and `Task` automatically receives a unique fingerprint when created, which cannot be manually overridden.

These fingerprints can be used for:

* Auditing and tracking component usage
* Ensuring component identity integrity
* Attaching metadata to components
* Creating a traceable chain of operations

## How Fingerprints Work

A fingerprint is an instance of the `Fingerprint` class from the `crewai.security` module. Each fingerprint contains:

* A UUID string: A unique identifier for the component that is automatically generated and cannot be manually set
* A creation timestamp: When the fingerprint was generated, automatically set and cannot be manually modified
* Metadata: A dictionary of additional information that can be customized

Fingerprints are automatically generated and assigned when a component is created. Each component exposes its fingerprint through a read-only property.

## Basic Usage

### Accessing Fingerprints

```python
from crewai import Agent, Crew, Task

# Create components - fingerprints are automatically generated
agent = Agent(
    role="Data Scientist",
    goal="Analyze data",
    backstory="Expert in data analysis"
)

crew = Crew(
    agents=[agent],
    tasks=[]
)

task = Task(
    description="Analyze customer data",
    expected_output="Insights from data analysis",
    agent=agent
)

# Access the fingerprints
agent_fingerprint = agent.fingerprint
crew_fingerprint = crew.fingerprint
task_fingerprint = task.fingerprint

# Print the UUID strings
print(f"Agent fingerprint: {agent_fingerprint.uuid_str}")
print(f"Crew fingerprint: {crew_fingerprint.uuid_str}")
print(f"Task fingerprint: {task_fingerprint.uuid_str}")
```

### Working with Fingerprint Metadata

You can add metadata to fingerprints for additional context:

```python
# Add metadata to the agent's fingerprint
agent.security_config.fingerprint.metadata = {
    "version": "1.0",
    "department": "Data Science",
    "project": "Customer Analysis"
}

# Access the metadata
print(f"Agent metadata: {agent.fingerprint.metadata}")
```

## Fingerprint Persistence

Fingerprints are designed to persist and remain unchanged throughout a component's lifecycle. If you modify a component, the fingerprint remains the same:

```python
original_fingerprint = agent.fingerprint.uuid_str

# Modify the agent
agent.goal = "New goal for analysis"

# The fingerprint remains unchanged
assert agent.fingerprint.uuid_str == original_fingerprint
```

## Deterministic Fingerprints

While you cannot directly set the UUID and creation timestamp, you can create deterministic fingerprints using the `generate` method with a seed:

```python
from crewai.security import Fingerprint

# Create a deterministic fingerprint using a seed string
deterministic_fingerprint = Fingerprint.generate(seed="my-agent-id")

# The same seed always produces the same fingerprint
same_fingerprint = Fingerprint.generate(seed="my-agent-id")
assert deterministic_fingerprint.uuid_str == same_fingerprint.uuid_str

# You can also set metadata
custom_fingerprint = Fingerprint.generate(
    seed="my-agent-id",
    metadata={"version": "1.0"}
)
```

## Advanced Usage

### Fingerprint Structure

Each fingerprint has the following structure:

```python
from crewai.security import Fingerprint

fingerprint = agent.fingerprint

# UUID string - the unique identifier (auto-generated)
uuid_str = fingerprint.uuid_str  # e.g., "123e4567-e89b-12d3-a456-426614174000"

# Creation timestamp (auto-generated)
created_at = fingerprint.created_at  # A datetime object

# Metadata - for additional information (can be customized)
metadata = fingerprint.metadata  # A dictionary, defaults to {}
```
