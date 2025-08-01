# Collaboration

> How to enable agents to work together, delegate tasks, and communicate effectively within CrewAI teams.

## Overview

Collaboration in CrewAI enables agents to work together as a team by delegating tasks and asking questions to leverage each other's expertise. When `allow_delegation=True`, agents automatically gain access to powerful collaboration tools.

## Quick Start: Enable Collaboration

```python
from crewai import Agent, Crew, Task

# Enable collaboration for agents
researcher = Agent(
    role="Research Specialist",
    goal="Conduct thorough research on any topic",
    backstory="Expert researcher with access to various sources",
    allow_delegation=True,  # 🔑 Key setting for collaboration
    verbose=True
)

writer = Agent(
    role="Content Writer", 
    goal="Create engaging content based on research",
    backstory="Skilled writer who transforms research into compelling content",
    allow_delegation=True,  # 🔑 Enables asking questions to other agents
    verbose=True
)

# Agents can now collaborate automatically
crew = Crew(
    agents=[researcher, writer],
    tasks=[...],
    verbose=True
)
```

## How Agent Collaboration Works

When `allow_delegation=True`, CrewAI automatically provides agents with two powerful tools:

### 1. **Delegate Work Tool**

Allows agents to assign tasks to teammates with specific expertise.

```python
# Agent automatically gets this tool:
# Delegate work to coworker(task: str, context: str, coworker: str)
```

### 2. **Ask Question Tool**

Enables agents to ask specific questions to gather information from colleagues.

```python
# Agent automatically gets this tool:
# Ask question to coworker(question: str, context: str, coworker: str)
```

## Collaboration in Action

Here's a complete example showing agents collaborating on a content creation task:

```python
from crewai import Agent, Crew, Task, Process

# Create collaborative agents
researcher = Agent(
    role="Research Specialist",
    goal="Find accurate, up-to-date information on any topic",
    backstory="""You're a meticulous researcher with expertise in finding 
    reliable sources and fact-checking information across various domains.""",
    allow_delegation=True,
    verbose=True
)

writer = Agent(
    role="Content Writer",
    goal="Create engaging, well-structured content",
    backstory="""You're a skilled content writer who excels at transforming 
    research into compelling, readable content for different audiences.""",
    allow_delegation=True,
    verbose=True
)

editor = Agent(
    role="Content Editor",
    goal="Ensure content quality and consistency",
    backstory="""You're an experienced editor with an eye for detail, 
    ensuring content meets high standards for clarity and accuracy.""",
    allow_delegation=True,
    verbose=True
)

# Create a task that encourages collaboration
article_task = Task(
    description="""Write a comprehensive 1000-word article about 'The Future of AI in Healthcare'.
    
    The article should include:
    - Current AI applications in healthcare
    - Emerging trends and technologies  
    - Potential challenges and ethical considerations
    - Expert predictions for the next 5 years
    
    Collaborate with your teammates to ensure accuracy and quality.""",
    expected_output="A well-researched, engaging 1000-word article with proper structure and citations",
    agent=writer  # Writer leads, but can delegate research to researcher
)

# Create collaborative crew
crew = Crew(
    agents=[researcher, writer, editor],
    tasks=[article_task],
    process=Process.sequential,
    verbose=True
)

result = crew.kickoff()
```

## Collaboration Patterns

### Pattern 1: Research → Write → Edit

```python
research_task = Task(
    description="Research the latest developments in quantum computing",
    expected_output="Comprehensive research summary with key findings and sources",
    agent=researcher
)

writing_task = Task(
    description="Write an article based on the research findings",
    expected_output="Engaging 800-word article about quantum computing",
    agent=writer,
    context=[research_task]  # Gets research output as context
)

editing_task = Task(
    description="Edit and polish the article for publication",
    expected_output="Publication-ready article with improved clarity and flow",
    agent=editor,
    context=[writing_task]  # Gets article draft as context
)
```

### Pattern 2: Collaborative Single Task

```python
collaborative_task = Task(
    description="""Create a marketing strategy for a new AI product.
    
    Writer: Focus on messaging and content strategy
    Researcher: Provide market analysis and competitor insights
    
    Work together to create a comprehensive strategy.""",
    expected_output="Complete marketing strategy with research backing",
    agent=writer  # Lead agent, but can delegate to researcher
)
```

## Hierarchical Collaboration

For complex projects, use a hierarchical process with a manager agent:

```python
from crewai import Agent, Crew, Task, Process

# Manager agent coordinates the team
manager = Agent(
    role="Project Manager",
    goal="Coordinate team efforts and ensure project success",
    backstory="Experienced project manager skilled at delegation and quality control",
    allow_delegation=True,
    verbose=True
)

# Specialist agents
researcher = Agent(
    role="Researcher",
    goal="Provide accurate research and analysis",
    backstory="Expert researcher with deep analytical skills",
    allow_delegation=False,  # Specialists focus on their expertise
    verbose=True
)

writer = Agent(
    role="Writer", 
    goal="Create compelling content",
    backstory="Skilled writer who creates engaging content",
    allow_delegation=False,
    verbose=True
)

# Manager-led task
project_task = Task(
    description="Create a comprehensive market analysis report with recommendations",
    expected_output="Executive summary, detailed analysis, and strategic recommendations",
    agent=manager  # Manager will delegate to specialists
)

# Hierarchical crew
crew = Crew(
    agents=[manager, researcher, writer],
    tasks=[project_task],
    process=Process.hierarchical,  # Manager coordinates everything
    manager_llm="gpt-4o",  # Specify LLM for manager
    verbose=True
)
```

## Best Practices for Collaboration

### 1. **Clear Role Definition**

```python
# ✅ Good: Specific, complementary roles
researcher = Agent(role="Market Research Analyst", ...)
writer = Agent(role="Technical Content Writer", ...)

# ❌ Avoid: Overlapping or vague roles  
agent1 = Agent(role="General Assistant", ...)
agent2 = Agent(role="Helper", ...)
```

### 2. **Strategic Delegation Enabling**

```python
# ✅ Enable delegation for coordinators and generalists
lead_agent = Agent(
    role="Content Lead",
    allow_delegation=True,  # Can delegate to specialists
    ...
)

# ✅ Disable for focused specialists (optional)
specialist_agent = Agent(
    role="Data Analyst", 
    allow_delegation=False,  # Focuses on core expertise
    ...
)
```

### 3. **Context Sharing**

```python
# ✅ Use context parameter for task dependencies
writing_task = Task(
    description="Write article based on research",
    agent=writer,
    context=[research_task],  # Shares research results
    ...
)
```

### 4. **Clear Task Descriptions**

```python
# ✅ Specific, actionable descriptions
Task(
    description="""Research competitors in the AI chatbot space.
    Focus on: pricing models, key features, target markets.
    Provide data in a structured format.""",
    ...
)

# ❌ Vague descriptions that don't guide collaboration
Task(description="Do some research about chatbots", ...)
```

## Troubleshooting Collaboration

### Issue: Agents Not Collaborating

**Symptoms:** Agents work in isolation, no delegation occurs

```python
# ✅ Solution: Ensure delegation is enabled
agent = Agent(
    role="...",
    allow_delegation=True,  # This is required!
    ...
)
```

### Issue: Too Much Back-and-Forth

**Symptoms:** Agents ask excessive questions, slow progress

```python
# ✅ Solution: Provide better context and specific roles
Task(
    description="""Write a technical blog post about machine learning.
    
    Context: Target audience is software developers with basic ML knowledge.
    Length: 1200 words
    Include: code examples, practical applications, best practices
    
    If you need specific technical details, delegate research to the researcher.""",
    ...
)
```

### Issue: Delegation Loops

**Symptoms:** Agents delegate back and forth indefinitely

```python
# ✅ Solution: Clear hierarchy and responsibilities
manager = Agent(role="Manager", allow_delegation=True)
specialist1 = Agent(role="Specialist A", allow_delegation=False)  # No re-delegation
specialist2 = Agent(role="Specialist B", allow_delegation=False)
```

## Advanced Collaboration Features

### Custom Collaboration Rules

```python
# Set specific collaboration guidelines in agent backstory
agent = Agent(
    role="Senior Developer",
    backstory="""You lead development projects and coordinate with team members.
    
    Collaboration guidelines:
    - Delegate research tasks to the Research Analyst
    - Ask the Designer for UI/UX guidance  
    - Consult the QA Engineer for testing strategies
    - Only escalate blocking issues to the Project Manager""",
    allow_delegation=True
)
```

### Monitoring Collaboration

```python
def track_collaboration(output):
    """Track collaboration patterns"""
    if "Delegate work to coworker" in output.raw:
        print("🤝 Delegation occurred")
    if "Ask question to coworker" in output.raw:
        print("❓ Question asked")

crew = Crew(
    agents=[...],
    tasks=[...],
    step_callback=track_collaboration,  # Monitor collaboration
    verbose=True
)
```

## Memory and Learning

Enable agents to remember past collaborations:

```python
agent = Agent(
    role="Content Lead",
    memory=True,  # Remembers past interactions
    allow_delegation=True,
    verbose=True
)
```

With memory enabled, agents learn from previous collaborations and improve their delegation decisions over time.

## Next Steps

* **Try the examples**: Start with the basic collaboration example
* **Experiment with roles**: Test different agent role combinations
* **Monitor interactions**: Use `verbose=True` to see collaboration in action
* **Optimize task descriptions**: Clear tasks lead to better collaboration
* **Scale up**: Try hierarchical processes for complex projects

Collaboration transforms individual AI agents into powerful teams that can tackle complex, multi-faceted challenges together.
