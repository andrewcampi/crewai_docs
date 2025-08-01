# Knowledge

> What is knowledge in CrewAI and how to use it.

## Overview

Knowledge in CrewAI is a powerful system that allows AI agents to access and utilize external information sources during their tasks.
Think of it as giving your agents a reference library they can consult while working.

<Info>
  Key benefits of using Knowledge:

  * Enhance agents with domain-specific information
  * Support decisions with real-world data
  * Maintain context across conversations
  * Ground responses in factual information
</Info>

## Quickstart Examples

<Tip>
  For file-based Knowledge Sources, make sure to place your files in a `knowledge` directory at the root of your project.
  Also, use relative paths from the `knowledge` directory when creating the source.
</Tip>

### Basic String Knowledge Example

```python Code
from crewai import Agent, Task, Crew, Process, LLM
from crewai.knowledge.source.string_knowledge_source import StringKnowledgeSource

# Create a knowledge source
content = "Users name is John. He is 30 years old and lives in San Francisco."
string_source = StringKnowledgeSource(content=content)

# Create an LLM with a temperature of 0 to ensure deterministic outputs
llm = LLM(model="gpt-4o-mini", temperature=0)

# Create an agent with the knowledge store
agent = Agent(
    role="About User",
    goal="You know everything about the user.",
    backstory="You are a master at understanding people and their preferences.",
    verbose=True,
    allow_delegation=False,
    llm=llm,
)

task = Task(
    description="Answer the following questions about the user: {question}",
    expected_output="An answer to the question.",
    agent=agent,
)

crew = Crew(
    agents=[agent],
    tasks=[task],
    verbose=True,
    process=Process.sequential,
    knowledge_sources=[string_source], # Enable knowledge by adding the sources here
)

result = crew.kickoff(inputs={"question": "What city does John live in and how old is he?"})
```

### Web Content Knowledge Example

<Note>
  You need to install `docling` for the following example to work: `uv add docling`
</Note>

```python Code
from crewai import LLM, Agent, Crew, Process, Task
from crewai.knowledge.source.crew_docling_source import CrewDoclingSource

# Create a knowledge source from web content
content_source = CrewDoclingSource(
    file_paths=[
        "https://lilianweng.github.io/posts/2024-11-28-reward-hacking",
        "https://lilianweng.github.io/posts/2024-07-07-hallucination",
    ],
)

# Create an LLM with a temperature of 0 to ensure deterministic outputs
llm = LLM(model="gpt-4o-mini", temperature=0)

# Create an agent with the knowledge store
agent = Agent(
    role="About papers",
    goal="You know everything about the papers.",
    backstory="You are a master at understanding papers and their content.",
    verbose=True,
    allow_delegation=False,
    llm=llm,
)

task = Task(
    description="Answer the following questions about the papers: {question}",
    expected_output="An answer to the question.",
    agent=agent,
)

crew = Crew(
    agents=[agent],
    tasks=[task],
    verbose=True,
    process=Process.sequential,
    knowledge_sources=[content_source],
)

result = crew.kickoff(
    inputs={"question": "What is the reward hacking paper about? Be sure to provide sources."}
)
```

## Supported Knowledge Sources

CrewAI supports various types of knowledge sources out of the box:

<CardGroup cols={2}>
  <Card title="Text Sources" icon="text">
    * Raw strings
    * Text files (.txt)
    * PDF documents
  </Card>

  <Card title="Structured Data" icon="table">
    * CSV files
    * Excel spreadsheets
    * JSON documents
  </Card>
</CardGroup>

### Text File Knowledge Source

```python
from crewai.knowledge.source.text_file_knowledge_source import TextFileKnowledgeSource

text_source = TextFileKnowledgeSource(
    file_paths=["document.txt", "another.txt"]
)
```

### PDF Knowledge Source

```python
from crewai.knowledge.source.pdf_knowledge_source import PDFKnowledgeSource

pdf_source = PDFKnowledgeSource(
    file_paths=["document.pdf", "another.pdf"]
)
```

### CSV Knowledge Source

```python
from crewai.knowledge.source.csv_knowledge_source import CSVKnowledgeSource

csv_source = CSVKnowledgeSource(
    file_paths=["data.csv"]
)
```

### Excel Knowledge Source

```python
from crewai.knowledge.source.excel_knowledge_source import ExcelKnowledgeSource

excel_source = ExcelKnowledgeSource(
    file_paths=["spreadsheet.xlsx"]
)
```

### JSON Knowledge Source

```python
from crewai.knowledge.source.json_knowledge_source import JSONKnowledgeSource

json_source = JSONKnowledgeSource(
    file_paths=["data.json"]
)
```

<Note>
  Please ensure that you create the ./knowledge folder. All source files (e.g., .txt, .pdf, .xlsx, .json) should be placed in this folder for centralized management.
</Note>

## Agent vs Crew Knowledge: Complete Guide

<Info>
  **Understanding Knowledge Levels**: CrewAI supports knowledge at both agent and crew levels. This section clarifies exactly how each works, when they're initialized, and addresses common misconceptions about dependencies.
</Info>

### How Knowledge Initialization Actually Works

Here's exactly what happens when you use knowledge:

#### Agent-Level Knowledge (Independent)

```python
from crewai import Agent, Task, Crew
from crewai.knowledge.source.string_knowledge_source import StringKnowledgeSource

# Agent with its own knowledge - NO crew knowledge needed
specialist_knowledge = StringKnowledgeSource(
    content="Specialized technical information for this agent only"
)

specialist_agent = Agent(
    role="Technical Specialist",
    goal="Provide technical expertise",
    backstory="Expert in specialized technical domains",
    knowledge_sources=[specialist_knowledge]  # Agent-specific knowledge
)

task = Task(
    description="Answer technical questions",
    agent=specialist_agent,
    expected_output="Technical answer"
)

# No crew-level knowledge required
crew = Crew(
    agents=[specialist_agent],
    tasks=[task]
)

result = crew.kickoff()  # Agent knowledge works independently
```

#### What Happens During `crew.kickoff()`

When you call `crew.kickoff()`, here's the exact sequence:

```python
# During kickoff
for agent in self.agents:
    agent.crew = self  # Agent gets reference to crew
    agent.set_knowledge(crew_embedder=self.embedder)  # Agent knowledge initialized
    agent.create_agent_executor()
```

#### Storage Independence

Each knowledge level uses independent storage collections:

```python
# Agent knowledge storage
agent_collection_name = agent.role  # e.g., "Technical Specialist"

# Crew knowledge storage  
crew_collection_name = "crew"

# Both stored in same ChromaDB instance but different collections
# Path: ~/.local/share/CrewAI/{project}/knowledge/
#   ├── crew/                    # Crew knowledge collection
#   ├── Technical Specialist/    # Agent knowledge collection
#   └── Another Agent Role/      # Another agent's collection
```

### Complete Working Examples

#### Example 1: Agent-Only Knowledge

```python
from crewai import Agent, Task, Crew
from crewai.knowledge.source.string_knowledge_source import StringKnowledgeSource

# Agent-specific knowledge
agent_knowledge = StringKnowledgeSource(
    content="Agent-specific information that only this agent needs"
)

agent = Agent(
    role="Specialist",
    goal="Use specialized knowledge",
    backstory="Expert with specific knowledge",
    knowledge_sources=[agent_knowledge],
    embedder={  # Agent can have its own embedder
        "provider": "openai",
        "config": {"model": "text-embedding-3-small"}
    }
)

task = Task(
    description="Answer using your specialized knowledge",
    agent=agent,
    expected_output="Answer based on agent knowledge"
)

# No crew knowledge needed
crew = Crew(agents=[agent], tasks=[task])
result = crew.kickoff()  # Works perfectly
```

#### Example 2: Both Agent and Crew Knowledge

```python
# Crew-wide knowledge (shared by all agents)
crew_knowledge = StringKnowledgeSource(
    content="Company policies and general information for all agents"
)

# Agent-specific knowledge
specialist_knowledge = StringKnowledgeSource(
    content="Technical specifications only the specialist needs"
)

specialist = Agent(
    role="Technical Specialist",
    goal="Provide technical expertise",
    backstory="Technical expert",
    knowledge_sources=[specialist_knowledge]  # Agent-specific
)

generalist = Agent(
    role="General Assistant", 
    goal="Provide general assistance",
    backstory="General helper"
    # No agent-specific knowledge
)

crew = Crew(
    agents=[specialist, generalist],
    tasks=[...],
    knowledge_sources=[crew_knowledge]  # Crew-wide knowledge
)

# Result:
# - specialist gets: crew_knowledge + specialist_knowledge
# - generalist gets: crew_knowledge only
```

#### Example 3: Multiple Agents with Different Knowledge

```python
# Different knowledge for different agents
sales_knowledge = StringKnowledgeSource(content="Sales procedures and pricing")
tech_knowledge = StringKnowledgeSource(content="Technical documentation")
support_knowledge = StringKnowledgeSource(content="Support procedures")

sales_agent = Agent(
    role="Sales Representative",
    knowledge_sources=[sales_knowledge],
    embedder={"provider": "openai", "config": {"model": "text-embedding-3-small"}}
)

tech_agent = Agent(
    role="Technical Expert", 
    knowledge_sources=[tech_knowledge],
    embedder={"provider": "ollama", "config": {"model": "mxbai-embed-large"}}
)

support_agent = Agent(
    role="Support Specialist",
    knowledge_sources=[support_knowledge]
    # Will use crew embedder as fallback
)

crew = Crew(
    agents=[sales_agent, tech_agent, support_agent],
    tasks=[...],
    embedder={  # Fallback embedder for agents without their own
        "provider": "google",
        "config": {"model": "text-embedding-004"}
    }
)

# Each agent gets only their specific knowledge
# Each can use different embedding providers
```

<Tip>
  Unlike retrieval from a vector database using a tool, agents preloaded with knowledge will not need a retrieval persona or task.
  Simply add the relevant knowledge sources your agent or crew needs to function.

  Knowledge sources can be added at the agent or crew level.
  Crew level knowledge sources will be used by **all agents** in the crew.
  Agent level knowledge sources will be used by the **specific agent** that is preloaded with the knowledge.
</Tip>

## Knowledge Configuration

You can configure the knowledge configuration for the crew or agent.

```python Code
from crewai.knowledge.knowledge_config import KnowledgeConfig

knowledge_config = KnowledgeConfig(results_limit=10, score_threshold=0.5)

agent = Agent(
    ...
    knowledge_config=knowledge_config
)
```

<Tip>
  `results_limit`: is the number of relevant documents to return. Default is 3.
  `score_threshold`: is the minimum score for a document to be considered relevant. Default is 0.35.
</Tip>

## Supported Knowledge Parameters

<ParamField body="sources" type="List[BaseKnowledgeSource]" required="Yes">
  List of knowledge sources that provide content to be stored and queried. Can include PDF, CSV, Excel, JSON, text files, or string content.
</ParamField>

<ParamField body="collection_name" type="str">
  Name of the collection where the knowledge will be stored. Used to identify different sets of knowledge. Defaults to "knowledge" if not provided.
</ParamField>

<ParamField body="storage" type="Optional[KnowledgeStorage]">
  Custom storage configuration for managing how the knowledge is stored and retrieved. If not provided, a default storage will be created.
</ParamField>

## Knowledge Storage Transparency

<Info>
  **Understanding Knowledge Storage**: CrewAI automatically stores knowledge sources in platform-specific directories using ChromaDB for vector storage. Understanding these locations and defaults helps with production deployments, debugging, and storage management.
</Info>

### Where CrewAI Stores Knowledge Files

By default, CrewAI uses the same storage system as memory, storing knowledge in platform-specific directories:

#### Default Storage Locations by Platform

**macOS:**

```
~/Library/Application Support/CrewAI/{project_name}/
└── knowledge/                    # Knowledge ChromaDB files
    ├── chroma.sqlite3           # ChromaDB metadata
    ├── {collection_id}/         # Vector embeddings
    └── knowledge_{collection}/  # Named collections
```

**Linux:**

```
~/.local/share/CrewAI/{project_name}/
└── knowledge/
    ├── chroma.sqlite3
    ├── {collection_id}/
    └── knowledge_{collection}/
```

**Windows:**

```
C:\Users\{username}\AppData\Local\CrewAI\{project_name}\
└── knowledge\
    ├── chroma.sqlite3
    ├── {collection_id}\
    └── knowledge_{collection}\
```

### Finding Your Knowledge Storage Location

To see exactly where CrewAI is storing your knowledge files:

```python
from crewai.utilities.paths import db_storage_path
import os

# Get the knowledge storage path
knowledge_path = os.path.join(db_storage_path(), "knowledge")
print(f"Knowledge storage location: {knowledge_path}")

# List knowledge collections and files
if os.path.exists(knowledge_path):
    print("\nKnowledge storage contents:")
    for item in os.listdir(knowledge_path):
        item_path = os.path.join(knowledge_path, item)
        if os.path.isdir(item_path):
            print(f"📁 Collection: {item}/")
            # Show collection contents
            try:
                for subitem in os.listdir(item_path):
                    print(f"   └── {subitem}")
            except PermissionError:
                print(f"   └── (permission denied)")
        else:
            print(f"📄 {item}")
else:
    print("No knowledge storage found yet.")
```

### Controlling Knowledge Storage Locations

#### Option 1: Environment Variable (Recommended)

```python
import os
from crewai import Crew

# Set custom storage location for all CrewAI data
os.environ["CREWAI_STORAGE_DIR"] = "./my_project_storage"

# All knowledge will now be stored in ./my_project_storage/knowledge/
crew = Crew(
    agents=[...],
    tasks=[...],
    knowledge_sources=[...]
)
```

#### Option 2: Custom Knowledge Storage

```python
from crewai.knowledge.storage.knowledge_storage import KnowledgeStorage
from crewai.knowledge.source.string_knowledge_source import StringKnowledgeSource

# Create custom storage with specific embedder
custom_storage = KnowledgeStorage(
    embedder={
        "provider": "ollama",
        "config": {"model": "mxbai-embed-large"}
    },
    collection_name="my_custom_knowledge"
)

# Use with knowledge sources
knowledge_source = StringKnowledgeSource(
    content="Your knowledge content here"
)
knowledge_source.storage = custom_storage
```

#### Option 3: Project-Specific Knowledge Storage

```python
import os
from pathlib import Path

# Store knowledge in project directory
project_root = Path(__file__).parent
knowledge_dir = project_root / "knowledge_storage"

os.environ["CREWAI_STORAGE_DIR"] = str(knowledge_dir)

# Now all knowledge will be stored in your project directory
```

### Default Embedding Provider Behavior

<Info>
  **Default Embedding Provider**: CrewAI defaults to OpenAI embeddings (`text-embedding-3-small`) for knowledge storage, even when using different LLM providers. You can easily customize this to match your setup.
</Info>

#### Understanding Default Behavior

```python
from crewai import Agent, Crew, LLM
from crewai.knowledge.source.string_knowledge_source import StringKnowledgeSource

# When using Claude as your LLM...
agent = Agent(
    role="Researcher",
    goal="Research topics",
    backstory="Expert researcher",
    llm=LLM(provider="anthropic", model="claude-3-sonnet")  # Using Claude
)

# CrewAI will still use OpenAI embeddings by default for knowledge
# This ensures consistency but may not match your LLM provider preference
knowledge_source = StringKnowledgeSource(content="Research data...")

crew = Crew(
    agents=[agent],
    tasks=[...],
    knowledge_sources=[knowledge_source]
    # Default: Uses OpenAI embeddings even with Claude LLM
)
```

#### Customizing Knowledge Embedding Providers

```python
# Option 1: Use Voyage AI (recommended by Anthropic for Claude users)
crew = Crew(
    agents=[agent],
    tasks=[...],
    knowledge_sources=[knowledge_source],
    embedder={
        "provider": "voyageai",  # Recommended for Claude users
        "config": {
            "api_key": "your-voyage-api-key",
            "model": "voyage-3"  # or "voyage-3-large" for best quality
        }
    }
)

# Option 2: Use local embeddings (no external API calls)
crew = Crew(
    agents=[agent],
    tasks=[...],
    knowledge_sources=[knowledge_source],
    embedder={
        "provider": "ollama",
        "config": {
            "model": "mxbai-embed-large",
            "url": "http://localhost:11434/api/embeddings"
        }
    }
)

# Option 3: Agent-level embedding customization
agent = Agent(
    role="Researcher",
    goal="Research topics",
    backstory="Expert researcher",
    knowledge_sources=[knowledge_source],
    embedder={
        "provider": "google",
        "config": {
            "model": "models/text-embedding-004",
            "api_key": "your-google-key"
        }
    }
)
```

#### Configuring Azure OpenAI Embeddings

When using Azure OpenAI embeddings:

1. Make sure you deploy the embedding model in Azure platform first
2. Then you need to use the following configuration:

```python
agent = Agent(
    role="Researcher",
    goal="Research topics",
    backstory="Expert researcher",
    knowledge_sources=[knowledge_source],
    embedder={
        "provider": "azure",
        "config": {
            "api_key": "your-azure-api-key",
            "model": "text-embedding-ada-002", # change to the model you are using and is deployed in Azure
            "api_base": "https://your-azure-endpoint.openai.azure.com/",
            "api_version": "2024-02-01"
        }
    }
)
```

## Advanced Features

### Query Rewriting

CrewAI implements an intelligent query rewriting mechanism to optimize knowledge retrieval. When an agent needs to search through knowledge sources, the raw task prompt is automatically transformed into a more effective search query.

#### How Query Rewriting Works

1. When an agent executes a task with knowledge sources available, the `_get_knowledge_search_query` method is triggered
2. The agent's LLM is used to transform the original task prompt into an optimized search query
3. This optimized query is then used to retrieve relevant information from knowledge sources

#### Benefits of Query Rewriting

<CardGroup cols={2}>
  <Card title="Improved Retrieval Accuracy" icon="bullseye-arrow">
    By focusing on key concepts and removing irrelevant content, query rewriting helps retrieve more relevant information.
  </Card>

  <Card title="Context Awareness" icon="brain">
    The rewritten queries are designed to be more specific and context-aware for vector database retrieval.
  </Card>
</CardGroup>

#### Example

```python
# Original task prompt
task_prompt = "Answer the following questions about the user's favorite movies: What movie did John watch last week? Format your answer in JSON."

# Behind the scenes, this might be rewritten as:
rewritten_query = "What movies did John watch last week?"
```

The rewritten query is more focused on the core information need and removes irrelevant instructions about output formatting.

<Tip>
  This mechanism is fully automatic and requires no configuration from users. The agent's LLM is used to perform the query rewriting, so using a more capable LLM can improve the quality of rewritten queries.
</Tip>

### Knowledge Events

CrewAI emits events during the knowledge retrieval process that you can listen for using the event system. These events allow you to monitor, debug, and analyze how knowledge is being retrieved and used by your agents.

#### Available Knowledge Events

* **KnowledgeRetrievalStartedEvent**: Emitted when an agent starts retrieving knowledge from sources
* **KnowledgeRetrievalCompletedEvent**: Emitted when knowledge retrieval is completed, including the query used and the retrieved content
* **KnowledgeQueryStartedEvent**: Emitted when a query to knowledge sources begins
* **KnowledgeQueryCompletedEvent**: Emitted when a query completes successfully
* **KnowledgeQueryFailedEvent**: Emitted when a query to knowledge sources fails
* **KnowledgeSearchQueryFailedEvent**: Emitted when a search query fails

#### Example: Monitoring Knowledge Retrieval

```python
from crewai.utilities.events import (
    KnowledgeRetrievalStartedEvent,
    KnowledgeRetrievalCompletedEvent,
)
from crewai.utilities.events.base_event_listener import BaseEventListener

class KnowledgeMonitorListener(BaseEventListener):
    def setup_listeners(self, crewai_event_bus):
        @crewai_event_bus.on(KnowledgeRetrievalStartedEvent)
        def on_knowledge_retrieval_started(source, event):
            print(f"Agent '{event.agent.role}' started retrieving knowledge")
            
        @crewai_event_bus.on(KnowledgeRetrievalCompletedEvent)
        def on_knowledge_retrieval_completed(source, event):
            print(f"Agent '{event.agent.role}' completed knowledge retrieval")
            print(f"Query: {event.query}")
            print(f"Retrieved {len(event.retrieved_knowledge)} knowledge chunks")

# Create an instance of your listener
knowledge_monitor = KnowledgeMonitorListener()
```

For more information on using events, see the [Event Listeners](https://docs.crewai.com/concepts/event-listener) documentation.

### Custom Knowledge Sources

CrewAI allows you to create custom knowledge sources for any type of data by extending the `BaseKnowledgeSource` class. Let's create a practical example that fetches and processes space news articles.

#### Space News Knowledge Source Example

<CodeGroup>
  ```python Code
  from crewai import Agent, Task, Crew, Process, LLM
  from crewai.knowledge.source.base_knowledge_source import BaseKnowledgeSource
  import requests
  from datetime import datetime
  from typing import Dict, Any
  from pydantic import BaseModel, Field

  class SpaceNewsKnowledgeSource(BaseKnowledgeSource):
      """Knowledge source that fetches data from Space News API."""

      api_endpoint: str = Field(description="API endpoint URL")
      limit: int = Field(default=10, description="Number of articles to fetch")

      def load_content(self) -> Dict[Any, str]:
          """Fetch and format space news articles."""
          try:
              response = requests.get(
                  f"{self.api_endpoint}?limit={self.limit}"
              )
              response.raise_for_status()

              data = response.json()
              articles = data.get('results', [])

              formatted_data = self.validate_content(articles)
              return {self.api_endpoint: formatted_data}
          except Exception as e:
              raise ValueError(f"Failed to fetch space news: {str(e)}")

      def validate_content(self, articles: list) -> str:
          """Format articles into readable text."""
          formatted = "Space News Articles:\n\n"
          for article in articles:
              formatted += f"""
                  Title: {article['title']}
                  Published: {article['published_at']}
                  Summary: {article['summary']}
                  News Site: {article['news_site']}
                  URL: {article['url']}
                  -------------------"""
          return formatted

      def add(self) -> None:
          """Process and store the articles."""
          content = self.load_content()
          for _, text in content.items():
              chunks = self._chunk_text(text)
              self.chunks.extend(chunks)

          self._save_documents()

  # Create knowledge source
  recent_news = SpaceNewsKnowledgeSource(
      api_endpoint="https://api.spaceflightnewsapi.net/v4/articles",
      limit=10,
  )

  # Create specialized agent
  space_analyst = Agent(
      role="Space News Analyst",
      goal="Answer questions about space news accurately and comprehensively",
      backstory="""You are a space industry analyst with expertise in space exploration,
      satellite technology, and space industry trends. You excel at answering questions
      about space news and providing detailed, accurate information.""",
      knowledge_sources=[recent_news],
      llm=LLM(model="gpt-4", temperature=0.0)
  )

  # Create task that handles user questions
  analysis_task = Task(
      description="Answer this question about space news: {user_question}",
      expected_output="A detailed answer based on the recent space news articles",
      agent=space_analyst
  )

  # Create and run the crew
  crew = Crew(
      agents=[space_analyst],
      tasks=[analysis_task],
      verbose=True,
      process=Process.sequential
  )

  # Example usage
  result = crew.kickoff(
      inputs={"user_question": "What are the latest developments in space exploration?"}
  )
  ```

  ```output Output
  # Agent: Space News Analyst
  ## Task: Answer this question about space news: What are the latest developments in space exploration?


  # Agent: Space News Analyst
  ## Final Answer:
  The latest developments in space exploration, based on recent space news articles, include the following:

  1. SpaceX has received the final regulatory approvals to proceed with the second integrated Starship/Super Heavy launch, scheduled for as soon as the morning of Nov. 17, 2023. This is a significant step in SpaceX's ambitious plans for space exploration and colonization. [Source: SpaceNews](https://spacenews.com/starship-cleared-for-nov-17-launch/)

  2. SpaceX has also informed the US Federal Communications Commission (FCC) that it plans to begin launching its first next-generation Starlink Gen2 satellites. This represents a major upgrade to the Starlink satellite internet service, which aims to provide high-speed internet access worldwide. [Source: Teslarati](https://www.teslarati.com/spacex-first-starlink-gen2-satellite-launch-2022/)

  3. AI startup Synthetaic has raised $15 million in Series B funding. The company uses artificial intelligence to analyze data from space and air sensors, which could have significant applications in space exploration and satellite technology. [Source: SpaceNews](https://spacenews.com/ai-startup-synthetaic-raises-15-million-in-series-b-funding/)

  4. The Space Force has formally established a unit within the U.S. Indo-Pacific Command, marking a permanent presence in the Indo-Pacific region. This could have significant implications for space security and geopolitics. [Source: SpaceNews](https://spacenews.com/space-force-establishes-permanent-presence-in-indo-pacific-region/)

  5. Slingshot Aerospace, a space tracking and data analytics company, is expanding its network of ground-based optical telescopes to increase coverage of low Earth orbit. This could improve our ability to track and analyze objects in low Earth orbit, including satellites and space debris. [Source: SpaceNews](https://spacenews.com/slingshots-space-tracking-network-to-extend-coverage-of-low-earth-orbit/)

  6. The National Natural Science Foundation of China has outlined a five-year project for researchers to study the assembly of ultra-large spacecraft. This could lead to significant advancements in spacecraft technology and space exploration capabilities. [Source: SpaceNews](https://spacenews.com/china-researching-challenges-of-kilometer-scale-ultra-large-spacecraft/)

  7. The Center for AEroSpace Autonomy Research (CAESAR) at Stanford University is focusing on spacecraft autonomy. The center held a kickoff event on May 22, 2024, to highlight the industry, academia, and government collaboration it seeks to foster. This could lead to significant advancements in autonomous spacecraft technology. [Source: SpaceNews](https://spacenews.com/stanford-center-focuses-on-spacecraft-autonomy/)
  ```
</CodeGroup>

## Debugging and Troubleshooting

### Debugging Knowledge Issues

#### Check Agent Knowledge Initialization

```python
from crewai import Agent, Crew, Task
from crewai.knowledge.source.string_knowledge_source import StringKnowledgeSource

knowledge_source = StringKnowledgeSource(content="Test knowledge")

agent = Agent(
    role="Test Agent",
    goal="Test knowledge",
    backstory="Testing",
    knowledge_sources=[knowledge_source]
)

crew = Crew(agents=[agent], tasks=[Task(...)])

# Before kickoff - knowledge not initialized
print(f"Before kickoff - Agent knowledge: {getattr(agent, 'knowledge', None)}")

crew.kickoff()

# After kickoff - knowledge initialized
print(f"After kickoff - Agent knowledge: {agent.knowledge}")
print(f"Agent knowledge collection: {agent.knowledge.storage.collection_name}")
print(f"Number of sources: {len(agent.knowledge.sources)}")
```

#### Verify Knowledge Storage Locations

```python
import os
from crewai.utilities.paths import db_storage_path

# Check storage structure
storage_path = db_storage_path()
knowledge_path = os.path.join(storage_path, "knowledge")

if os.path.exists(knowledge_path):
    print("Knowledge collections found:")
    for collection in os.listdir(knowledge_path):
        collection_path = os.path.join(knowledge_path, collection)
        if os.path.isdir(collection_path):
            print(f"  - {collection}/")
            # Show collection contents
            for item in os.listdir(collection_path):
                print(f"    └── {item}")
```

#### Test Knowledge Retrieval

```python
# Test agent knowledge retrieval
if hasattr(agent, 'knowledge') and agent.knowledge:
    test_query = ["test query"]
    results = agent.knowledge.query(test_query)
    print(f"Agent knowledge results: {len(results)} documents found")
    
    # Test crew knowledge retrieval (if exists)
    if hasattr(crew, 'knowledge') and crew.knowledge:
        crew_results = crew.query_knowledge(test_query)
        print(f"Crew knowledge results: {len(crew_results)} documents found")
```

#### Inspect Knowledge Collections

```python
import chromadb
from crewai.utilities.paths import db_storage_path
import os

# Connect to CrewAI's knowledge ChromaDB
knowledge_path = os.path.join(db_storage_path(), "knowledge")

if os.path.exists(knowledge_path):
    client = chromadb.PersistentClient(path=knowledge_path)
    collections = client.list_collections()
    
    print("Knowledge Collections:")
    for collection in collections:
        print(f"  - {collection.name}: {collection.count()} documents")
        
        # Sample a few documents to verify content
        if collection.count() > 0:
            sample = collection.peek(limit=2)
            print(f"    Sample content: {sample['documents'][0][:100]}...")
else:
    print("No knowledge storage found")
```

#### Check Knowledge Processing

```python
from crewai.knowledge.source.string_knowledge_source import StringKnowledgeSource

# Create a test knowledge source
test_source = StringKnowledgeSource(
    content="Test knowledge content for debugging",
    chunk_size=100,  # Small chunks for testing
    chunk_overlap=20
)

# Check chunking behavior
print(f"Original content length: {len(test_source.content)}")
print(f"Chunk size: {test_source.chunk_size}")
print(f"Chunk overlap: {test_source.chunk_overlap}")

# Process and inspect chunks
test_source.add()
print(f"Number of chunks created: {len(test_source.chunks)}")
for i, chunk in enumerate(test_source.chunks[:3]):  # Show first 3 chunks
    print(f"Chunk {i+1}: {chunk[:50]}...")
```

### Common Knowledge Storage Issues

**"File not found" errors:**

```python
# Ensure files are in the correct location
from crewai.utilities.constants import KNOWLEDGE_DIRECTORY
import os

knowledge_dir = KNOWLEDGE_DIRECTORY  # Usually "knowledge"
file_path = os.path.join(knowledge_dir, "your_file.pdf")

if not os.path.exists(file_path):
    print(f"File not found: {file_path}")
    print(f"Current working directory: {os.getcwd()}")
    print(f"Expected knowledge directory: {os.path.abspath(knowledge_dir)}")
```

**"Embedding dimension mismatch" errors:**

```python
# This happens when switching embedding providers
# Reset knowledge storage to clear old embeddings
crew.reset_memories(command_type='knowledge')

# Or use consistent embedding providers
crew = Crew(
    agents=[...],
    tasks=[...],
    knowledge_sources=[...],
    embedder={"provider": "openai", "config": {"model": "text-embedding-3-small"}}
)
```

**"ChromaDB permission denied" errors:**

```bash
# Fix storage permissions
chmod -R 755 ~/.local/share/CrewAI/
```

**Knowledge not persisting between runs:**

```python
# Verify storage location consistency
import os
from crewai.utilities.paths import db_storage_path

print("CREWAI_STORAGE_DIR:", os.getenv("CREWAI_STORAGE_DIR"))
print("Computed storage path:", db_storage_path())
print("Knowledge path:", os.path.join(db_storage_path(), "knowledge"))
```

### Knowledge Reset Commands

```python
# Reset only agent-specific knowledge
crew.reset_memories(command_type='agent_knowledge')

# Reset both crew and agent knowledge  
crew.reset_memories(command_type='knowledge')

# CLI commands
# crewai reset-memories --agent-knowledge  # Agent knowledge only
# crewai reset-memories --knowledge        # All knowledge
```

### Clearing Knowledge

If you need to clear the knowledge stored in CrewAI, you can use the `crewai reset-memories` command with the `--knowledge` option.

```bash Command
crewai reset-memories --knowledge
```

This is useful when you've updated your knowledge sources and want to ensure that the agents are using the most recent information.

## Best Practices

<AccordionGroup>
  <Accordion title="Content Organization">
    * Keep chunk sizes appropriate for your content type
    * Consider content overlap for context preservation
    * Organize related information into separate knowledge sources
  </Accordion>

  <Accordion title="Performance Tips">
    * Adjust chunk sizes based on content complexity
    * Configure appropriate embedding models
    * Consider using local embedding providers for faster processing
  </Accordion>

  <Accordion title="One Time Knowledge">
    * With the typical file structure provided by CrewAI, knowledge sources are embedded every time the kickoff is triggered.
    * If the knowledge sources are large, this leads to inefficiency and increased latency, as the same data is embedded each time.
    * To resolve this, directly initialize the knowledge parameter instead of the knowledge\_sources parameter.
    * Link to the issue to get complete idea [Github Issue](https://github.com/crewAIInc/crewAI/issues/2755)
  </Accordion>

  <Accordion title="Knowledge Management">
    * Use agent-level knowledge for role-specific information
    * Use crew-level knowledge for shared information all agents need
    * Set embedders at agent level if you need different embedding strategies
    * Use consistent collection naming by keeping agent roles descriptive
    * Test knowledge initialization by checking agent.knowledge after kickoff
    * Monitor storage locations to understand where knowledge is stored
    * Reset knowledge appropriately using the correct command types
  </Accordion>

  <Accordion title="Production Best Practices">
    * Set `CREWAI_STORAGE_DIR` to a known location in production
    * Choose explicit embedding providers to match your LLM setup and avoid API key conflicts
    * Monitor knowledge storage size as it grows with document additions
    * Organize knowledge sources by domain or purpose using collection names
    * Include knowledge directories in your backup and deployment strategies
    * Set appropriate file permissions for knowledge files and storage directories
    * Use environment variables for API keys and sensitive configuration
  </Accordion>
</AccordionGroup>
