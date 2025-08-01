# Tools

> Understanding and leveraging tools within the CrewAI framework for agent collaboration and task execution.

## Overview

CrewAI tools empower agents with capabilities ranging from web searching and data analysis to collaboration and delegating tasks among coworkers.
This documentation outlines how to create, integrate, and leverage these tools within the CrewAI framework, including a new focus on collaboration tools.

## What is a Tool?

A tool in CrewAI is a skill or function that agents can utilize to perform various actions.
This includes tools from the [CrewAI Toolkit](https://github.com/joaomdmoura/crewai-tools) and [LangChain Tools](https://python.langchain.com/docs/integrations/tools),
enabling everything from simple searches to complex interactions and effective teamwork among agents.

<Note type="info" title="Enterprise Enhancement: Tools Repository">
  CrewAI Enterprise provides a comprehensive Tools Repository with pre-built integrations for common business systems and APIs. Deploy agents with enterprise tools in minutes instead of days.

  The Enterprise Tools Repository includes:

  * Pre-built connectors for popular enterprise systems
  * Custom tool creation interface
  * Version control and sharing capabilities
  * Security and compliance features
</Note>

## Key Characteristics of Tools

* **Utility**: Crafted for tasks such as web searching, data analysis, content generation, and agent collaboration.
* **Integration**: Boosts agent capabilities by seamlessly integrating tools into their workflow.
* **Customizability**: Provides the flexibility to develop custom tools or utilize existing ones, catering to the specific needs of agents.
* **Error Handling**: Incorporates robust error handling mechanisms to ensure smooth operation.
* **Caching Mechanism**: Features intelligent caching to optimize performance and reduce redundant operations.
* **Asynchronous Support**: Handles both synchronous and asynchronous tools, enabling non-blocking operations.

## Using CrewAI Tools

To enhance your agents' capabilities with crewAI tools, begin by installing our extra tools package:

```bash
pip install 'crewai[tools]'
```

Here's an example demonstrating their use:

```python Code
import os
from crewai import Agent, Task, Crew
# Importing crewAI tools
from crewai_tools import (
    DirectoryReadTool,
    FileReadTool,
    SerperDevTool,
    WebsiteSearchTool
)

# Set up API keys
os.environ["SERPER_API_KEY"] = "Your Key" # serper.dev API key
os.environ["OPENAI_API_KEY"] = "Your Key"

# Instantiate tools
docs_tool = DirectoryReadTool(directory='./blog-posts')
file_tool = FileReadTool()
search_tool = SerperDevTool()
web_rag_tool = WebsiteSearchTool()

# Create agents
researcher = Agent(
    role='Market Research Analyst',
    goal='Provide up-to-date market analysis of the AI industry',
    backstory='An expert analyst with a keen eye for market trends.',
    tools=[search_tool, web_rag_tool],
    verbose=True
)

writer = Agent(
    role='Content Writer',
    goal='Craft engaging blog posts about the AI industry',
    backstory='A skilled writer with a passion for technology.',
    tools=[docs_tool, file_tool],
    verbose=True
)

# Define tasks
research = Task(
    description='Research the latest trends in the AI industry and provide a summary.',
    expected_output='A summary of the top 3 trending developments in the AI industry with a unique perspective on their significance.',
    agent=researcher
)

write = Task(
    description='Write an engaging blog post about the AI industry, based on the research analyst's summary. Draw inspiration from the latest blog posts in the directory.',
    expected_output='A 4-paragraph blog post formatted in markdown with engaging, informative, and accessible content, avoiding complex jargon.',
    agent=writer,
    output_file='blog-posts/new_post.md'  # The final blog post will be saved here
)

# Assemble a crew with planning enabled
crew = Crew(
    agents=[researcher, writer],
    tasks=[research, write],
    verbose=True,
    planning=True,  # Enable planning feature
)

# Execute tasks
crew.kickoff()
```

## Available CrewAI Tools

* **Error Handling**: All tools are built with error handling capabilities, allowing agents to gracefully manage exceptions and continue their tasks.
* **Caching Mechanism**: All tools support caching, enabling agents to efficiently reuse previously obtained results, reducing the load on external resources and speeding up the execution time. You can also define finer control over the caching mechanism using the `cache_function` attribute on the tool.

Here is a list of the available tools and their descriptions:

| Tool                             | Description                                                                                    |
| :------------------------------- | :--------------------------------------------------------------------------------------------- |
| **ApifyActorsTool**              | A tool that integrates Apify Actors with your workflows for web scraping and automation tasks. |
| **BrowserbaseLoadTool**          | A tool for interacting with and extracting data from web browsers.                             |
| **CodeDocsSearchTool**           | A RAG tool optimized for searching through code documentation and related technical documents. |
| **CodeInterpreterTool**          | A tool for interpreting python code.                                                           |
| **ComposioTool**                 | Enables use of Composio tools.                                                                 |
| **CSVSearchTool**                | A RAG tool designed for searching within CSV files, tailored to handle structured data.        |
| **DALL-E Tool**                  | A tool for generating images using the DALL-E API.                                             |
| **DirectorySearchTool**          | A RAG tool for searching within directories, useful for navigating through file systems.       |
| **DOCXSearchTool**               | A RAG tool aimed at searching within DOCX documents, ideal for processing Word files.          |
| **DirectoryReadTool**            | Facilitates reading and processing of directory structures and their contents.                 |
| **EXASearchTool**                | A tool designed for performing exhaustive searches across various data sources.                |
| **FileReadTool**                 | Enables reading and extracting data from files, supporting various file formats.               |
| **FirecrawlSearchTool**          | A tool to search webpages using Firecrawl and return the results.                              |
| **FirecrawlCrawlWebsiteTool**    | A tool for crawling webpages using Firecrawl.                                                  |
| **FirecrawlScrapeWebsiteTool**   | A tool for scraping webpages URL using Firecrawl and returning its contents.                   |
| **GithubSearchTool**             | A RAG tool for searching within GitHub repositories, useful for code and documentation search. |
| **SerperDevTool**                | A specialized tool for development purposes, with specific functionalities under development.  |
| **TXTSearchTool**                | A RAG tool focused on searching within text (.txt) files, suitable for unstructured data.      |
| **JSONSearchTool**               | A RAG tool designed for searching within JSON files, catering to structured data handling.     |
| **LlamaIndexTool**               | Enables the use of LlamaIndex tools.                                                           |
| **MDXSearchTool**                | A RAG tool tailored for searching within Markdown (MDX) files, useful for documentation.       |
| **PDFSearchTool**                | A RAG tool aimed at searching within PDF documents, ideal for processing scanned documents.    |
| **PGSearchTool**                 | A RAG tool optimized for searching within PostgreSQL databases, suitable for database queries. |
| **Vision Tool**                  | A tool for generating images using the DALL-E API.                                             |
| **RagTool**                      | A general-purpose RAG tool capable of handling various data sources and types.                 |
| **ScrapeElementFromWebsiteTool** | Enables scraping specific elements from websites, useful for targeted data extraction.         |
| **ScrapeWebsiteTool**            | Facilitates scraping entire websites, ideal for comprehensive data collection.                 |
| **WebsiteSearchTool**            | A RAG tool for searching website content, optimized for web data extraction.                   |
| **XMLSearchTool**                | A RAG tool designed for searching within XML files, suitable for structured data formats.      |
| **YoutubeChannelSearchTool**     | A RAG tool for searching within YouTube channels, useful for video content analysis.           |
| **YoutubeVideoSearchTool**       | A RAG tool aimed at searching within YouTube videos, ideal for video data extraction.          |

## Creating your own Tools

<Tip>
  Developers can craft `custom tools` tailored for their agent's needs or
  utilize pre-built options.
</Tip>

There are two main ways for one to create a CrewAI tool:

### Subclassing `BaseTool`

```python Code
from crewai.tools import BaseTool
from pydantic import BaseModel, Field

class MyToolInput(BaseModel):
    """Input schema for MyCustomTool."""
    argument: str = Field(..., description="Description of the argument.")

class MyCustomTool(BaseTool):
    name: str = "Name of my tool"
    description: str = "What this tool does. It's vital for effective utilization."
    args_schema: Type[BaseModel] = MyToolInput

    def _run(self, argument: str) -> str:
        # Your tool's logic here
        return "Tool's result"
```

## Asynchronous Tool Support

CrewAI supports asynchronous tools, allowing you to implement tools that perform non-blocking operations like network requests, file I/O, or other async operations without blocking the main execution thread.

### Creating Async Tools

You can create async tools in two ways:

#### 1. Using the `tool` Decorator with Async Functions

```python Code
from crewai.tools import tool

@tool("fetch_data_async")
async def fetch_data_async(query: str) -> str:
    """Asynchronously fetch data based on the query."""
    # Simulate async operation
    await asyncio.sleep(1)
    return f"Data retrieved for {query}"
```

#### 2. Implementing Async Methods in Custom Tool Classes

```python Code
from crewai.tools import BaseTool

class AsyncCustomTool(BaseTool):
    name: str = "async_custom_tool"
    description: str = "An asynchronous custom tool"
    
    async def _run(self, query: str = "") -> str:
        """Asynchronously run the tool"""
        # Your async implementation here
        await asyncio.sleep(1)
        return f"Processed {query} asynchronously"
```

### Using Async Tools

Async tools work seamlessly in both standard Crew workflows and Flow-based workflows:

```python Code
# In standard Crew
agent = Agent(role="researcher", tools=[async_custom_tool])

# In Flow
class MyFlow(Flow):
    @start()
    async def begin(self):
        crew = Crew(agents=[agent])
        result = await crew.kickoff_async()
        return result
```

The CrewAI framework automatically handles the execution of both synchronous and asynchronous tools, so you don't need to worry about how to call them differently.

### Utilizing the `tool` Decorator

```python Code
from crewai.tools import tool
@tool("Name of my tool")
def my_tool(question: str) -> str:
    """Clear description for what this tool is useful for, your agent will need this information to use it."""
    # Function logic here
    return "Result from your custom tool"
```

### Custom Caching Mechanism

<Tip>
  Tools can optionally implement a `cache_function` to fine-tune caching
  behavior. This function determines when to cache results based on specific
  conditions, offering granular control over caching logic.
</Tip>

```python Code
from crewai.tools import tool

@tool
def multiplication_tool(first_number: int, second_number: int) -> str:
    """Useful for when you need to multiply two numbers together."""
    return first_number * second_number

def cache_func(args, result):
    # In this case, we only cache the result if it's a multiple of 2
    cache = result % 2 == 0
    return cache

multiplication_tool.cache_function = cache_func

writer1 = Agent(
        role="Writer",
        goal="You write lessons of math for kids.",
        backstory="You're an expert in writing and you love to teach kids but you know nothing of math.",
        tools=[multiplication_tool],
        allow_delegation=False,
    )
    #...
```

## Conclusion

Tools are pivotal in extending the capabilities of CrewAI agents, enabling them to undertake a broad spectrum of tasks and collaborate effectively.
When building solutions with CrewAI, leverage both custom and existing tools to empower your agents and enhance the AI ecosystem. Consider utilizing error handling,
caching mechanisms, and the flexibility of tool arguments to optimize your agents' performance and capabilities.
