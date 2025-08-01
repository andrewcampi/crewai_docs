# Using Annotations in crew.py

> Learn how to use annotations to properly structure agents, tasks, and components in CrewAI

This guide explains how to use annotations to properly reference **agents**, **tasks**, and other components in the `crew.py` file.

## Introduction

Annotations in the CrewAI framework are used to decorate classes and methods, providing metadata and functionality to various components of your crew. These annotations help in organizing and structuring your code, making it more readable and maintainable.

## Available Annotations

The CrewAI framework provides the following annotations:

* `@CrewBase`: Used to decorate the main crew class.
* `@agent`: Decorates methods that define and return Agent objects.
* `@task`: Decorates methods that define and return Task objects.
* `@crew`: Decorates the method that creates and returns the Crew object.
* `@llm`: Decorates methods that initialize and return Language Model objects.
* `@tool`: Decorates methods that initialize and return Tool objects.
* `@callback`: Used for defining callback methods.
* `@output_json`: Used for methods that output JSON data.
* `@output_pydantic`: Used for methods that output Pydantic models.
* `@cache_handler`: Used for defining cache handling methods.

## Usage Examples

Let's go through examples of how to use these annotations:

### 1. Crew Base Class

```python
@CrewBase
class LinkedinProfileCrew():
    """LinkedinProfile crew"""
    agents_config = 'config/agents.yaml'
    tasks_config = 'config/tasks.yaml'
```

The `@CrewBase` annotation is used to decorate the main crew class. This class typically contains configurations and methods for creating agents, tasks, and the crew itself.

### 2. Tool Definition

```python
@tool
def myLinkedInProfileTool(self):
    return LinkedInProfileTool()
```

The `@tool` annotation is used to decorate methods that return tool objects. These tools can be used by agents to perform specific tasks.

### 3. LLM Definition

```python
@llm
def groq_llm(self):
    api_key = os.getenv('api_key')
    return ChatGroq(api_key=api_key, temperature=0, model_name="mixtral-8x7b-32768")
```

The `@llm` annotation is used to decorate methods that initialize and return Language Model objects. These LLMs are used by agents for natural language processing tasks.

### 4. Agent Definition

```python
@agent
def researcher(self) -> Agent:
    return Agent(
        config=self.agents_config['researcher']
    )
```

The `@agent` annotation is used to decorate methods that define and return Agent objects.

### 5. Task Definition

```python
@task
def research_task(self) -> Task:
    return Task(
        config=self.tasks_config['research_linkedin_task'],
        agent=self.researcher()
    )
```

The `@task` annotation is used to decorate methods that define and return Task objects. These methods specify the task configuration and the agent responsible for the task.

### 6. Crew Creation

```python
@crew
def crew(self) -> Crew:
    """Creates the LinkedinProfile crew"""
    return Crew(
        agents=self.agents,
        tasks=self.tasks,
        process=Process.sequential,
        verbose=True
    )
```

The `@crew` annotation is used to decorate the method that creates and returns the `Crew` object. This method assembles all the components (agents and tasks) into a functional crew.

## YAML Configuration

The agent configurations are typically stored in a YAML file. Here's an example of how the `agents.yaml` file might look for the researcher agent:

```yaml
researcher:
    role: >
        LinkedIn Profile Senior Data Researcher
    goal: >
        Uncover detailed LinkedIn profiles based on provided name {name} and domain {domain}
        Generate a Dall-E image based on domain {domain}
    backstory: >
        You're a seasoned researcher with a knack for uncovering the most relevant LinkedIn profiles.
        Known for your ability to navigate LinkedIn efficiently, you excel at gathering and presenting
        professional information clearly and concisely.
    allow_delegation: False
    verbose: True
    llm: groq_llm
    tools:
        - myLinkedInProfileTool
        - mySerperDevTool
        - myDallETool
```

This YAML configuration corresponds to the researcher agent defined in the `LinkedinProfileCrew` class. The configuration specifies the agent's role, goal, backstory, and other properties such as the LLM and tools it uses.

Note how the `llm` and `tools` in the YAML file correspond to the methods decorated with `@llm` and `@tool` in the Python class.

## Best Practices

* **Consistent Naming**: Use clear and consistent naming conventions for your methods. For example, agent methods could be named after their roles (e.g., researcher, reporting\_analyst).
* **Environment Variables**: Use environment variables for sensitive information like API keys.
* **Flexibility**: Design your crew to be flexible by allowing easy addition or removal of agents and tasks.
* **YAML-Code Correspondence**: Ensure that the names and structures in your YAML files correspond correctly to the decorated methods in your Python code.

By following these guidelines and properly using annotations, you can create well-structured and maintainable crews using the CrewAI framework.
