# Memory

> Leveraging memory systems in the CrewAI framework to enhance agent capabilities.

## Overview

The CrewAI framework provides a sophisticated memory system designed to significantly enhance AI agent capabilities. CrewAI offers **three distinct memory approaches** that serve different use cases:

1. **Basic Memory System** - Built-in short-term, long-term, and entity memory
2. **User Memory** - User-specific memory with Mem0 integration (legacy approach)
3. **External Memory** - Standalone external memory providers (new approach)

## Memory System Components

| Component             | Description                                                                                                                                                                                                      |
| :-------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Short-Term Memory** | Temporarily stores recent interactions and outcomes using `RAG`, enabling agents to recall and utilize information relevant to their current context during the current executions.                              |
| **Long-Term Memory**  | Preserves valuable insights and learnings from past executions, allowing agents to build and refine their knowledge over time.                                                                                   |
| **Entity Memory**     | Captures and organizes information about entities (people, places, concepts) encountered during tasks, facilitating deeper understanding and relationship mapping. Uses `RAG` for storing entity information.    |
| **Contextual Memory** | Maintains the context of interactions by combining `ShortTermMemory`, `LongTermMemory`, and `EntityMemory`, aiding in the coherence and relevance of agent responses over a sequence of tasks or a conversation. |

## 1. Basic Memory System (Recommended)

The simplest and most commonly used approach. Enable memory for your crew with a single parameter:

### Quick Start

```python
from crewai import Crew, Agent, Task, Process

# Enable basic memory system
crew = Crew(
    agents=[...],
    tasks=[...],
    process=Process.sequential,
    memory=True,  # Enables short-term, long-term, and entity memory
    verbose=True
)
```

### How It Works

* **Short-Term Memory**: Uses ChromaDB with RAG for current context
* **Long-Term Memory**: Uses SQLite3 to store task results across sessions
* **Entity Memory**: Uses RAG to track entities (people, places, concepts)
* **Storage Location**: Platform-specific location via `appdirs` package
* **Custom Storage Directory**: Set `CREWAI_STORAGE_DIR` environment variable

## Storage Location Transparency

<Info>
  **Understanding Storage Locations**: CrewAI uses platform-specific directories to store memory and knowledge files following OS conventions. Understanding these locations helps with production deployments, backups, and debugging.
</Info>

### Where CrewAI Stores Files

By default, CrewAI uses the `appdirs` library to determine storage locations following platform conventions. Here's exactly where your files are stored:

#### Default Storage Locations by Platform

**macOS:**

```
~/Library/Application Support/CrewAI/{project_name}/
├── knowledge/           # Knowledge base ChromaDB files
├── short_term_memory/   # Short-term memory ChromaDB files
├── long_term_memory/    # Long-term memory ChromaDB files
├── entities/            # Entity memory ChromaDB files
└── long_term_memory_storage.db  # SQLite database
```

**Linux:**

```
~/.local/share/CrewAI/{project_name}/
├── knowledge/
├── short_term_memory/
├── long_term_memory/
├── entities/
└── long_term_memory_storage.db
```

**Windows:**

```
C:\Users\{username}\AppData\Local\CrewAI\{project_name}\
├── knowledge\
├── short_term_memory\
├── long_term_memory\
├── entities\
└── long_term_memory_storage.db
```

### Finding Your Storage Location

To see exactly where CrewAI is storing files on your system:

```python
from crewai.utilities.paths import db_storage_path
import os

# Get the base storage path
storage_path = db_storage_path()
print(f"CrewAI storage location: {storage_path}")

# List all CrewAI storage directories
if os.path.exists(storage_path):
    print("\nStored files and directories:")
    for item in os.listdir(storage_path):
        item_path = os.path.join(storage_path, item)
        if os.path.isdir(item_path):
            print(f"📁 {item}/")
            # Show ChromaDB collections
            if os.path.exists(item_path):
                for subitem in os.listdir(item_path):
                    print(f"   └── {subitem}")
        else:
            print(f"📄 {item}")
else:
    print("No CrewAI storage directory found yet.")
```

### Controlling Storage Locations

#### Option 1: Environment Variable (Recommended)

```python
import os
from crewai import Crew

# Set custom storage location
os.environ["CREWAI_STORAGE_DIR"] = "./my_project_storage"

# All memory and knowledge will now be stored in ./my_project_storage/
crew = Crew(
    agents=[...],
    tasks=[...],
    memory=True
)
```

#### Option 2: Custom Storage Paths

```python
import os
from crewai import Crew
from crewai.memory import LongTermMemory
from crewai.memory.storage.ltm_sqlite_storage import LTMSQLiteStorage

# Configure custom storage location
custom_storage_path = "./storage"
os.makedirs(custom_storage_path, exist_ok=True)

crew = Crew(
    memory=True,
    long_term_memory=LongTermMemory(
        storage=LTMSQLiteStorage(
            db_path=f"{custom_storage_path}/memory.db"
        )
    )
)
```

#### Option 3: Project-Specific Storage

```python
import os
from pathlib import Path

# Store in project directory
project_root = Path(__file__).parent
storage_dir = project_root / "crewai_storage"

os.environ["CREWAI_STORAGE_DIR"] = str(storage_dir)

# Now all storage will be in your project directory
```

### Embedding Provider Defaults

<Info>
  **Default Embedding Provider**: CrewAI defaults to OpenAI embeddings for consistency and reliability. You can easily customize this to match your LLM provider or use local embeddings.
</Info>

#### Understanding Default Behavior

```python
# When using Claude as your LLM...
from crewai import Agent, LLM

agent = Agent(
    role="Analyst",
    goal="Analyze data",
    backstory="Expert analyst",
    llm=LLM(provider="anthropic", model="claude-3-sonnet")  # Using Claude
)

# CrewAI will use OpenAI embeddings by default for consistency
# You can easily customize this to match your preferred provider
```

#### Customizing Embedding Providers

```python
from crewai import Crew

# Option 1: Match your LLM provider
crew = Crew(
    agents=[agent],
    tasks=[task],
    memory=True,
    embedder={
        "provider": "anthropic",  # Match your LLM provider
        "config": {
            "api_key": "your-anthropic-key",
            "model": "text-embedding-3-small"
        }
    }
)

# Option 2: Use local embeddings (no external API calls)
crew = Crew(
    agents=[agent],
    tasks=[task],
    memory=True,
    embedder={
        "provider": "ollama",
        "config": {"model": "mxbai-embed-large"}
    }
)
```

### Debugging Storage Issues

#### Check Storage Permissions

```python
import os
from crewai.utilities.paths import db_storage_path

storage_path = db_storage_path()
print(f"Storage path: {storage_path}")
print(f"Path exists: {os.path.exists(storage_path)}")
print(f"Is writable: {os.access(storage_path, os.W_OK) if os.path.exists(storage_path) else 'Path does not exist'}")

# Create with proper permissions
if not os.path.exists(storage_path):
    os.makedirs(storage_path, mode=0o755, exist_ok=True)
    print(f"Created storage directory: {storage_path}")
```

#### Inspect ChromaDB Collections

```python
import chromadb
from crewai.utilities.paths import db_storage_path

# Connect to CrewAI's ChromaDB
storage_path = db_storage_path()
chroma_path = os.path.join(storage_path, "knowledge")

if os.path.exists(chroma_path):
    client = chromadb.PersistentClient(path=chroma_path)
    collections = client.list_collections()

    print("ChromaDB Collections:")
    for collection in collections:
        print(f"  - {collection.name}: {collection.count()} documents")
else:
    print("No ChromaDB storage found")
```

#### Reset Storage (Debugging)

```python
from crewai import Crew

# Reset all memory storage
crew = Crew(agents=[...], tasks=[...], memory=True)

# Reset specific memory types
crew.reset_memories(command_type='short')     # Short-term memory
crew.reset_memories(command_type='long')      # Long-term memory
crew.reset_memories(command_type='entity')    # Entity memory
crew.reset_memories(command_type='knowledge') # Knowledge storage
```

### Production Best Practices

1. **Set `CREWAI_STORAGE_DIR`** to a known location in production for better control
2. **Choose explicit embedding providers** to match your LLM setup
3. **Monitor storage directory size** for large-scale deployments
4. **Include storage directories** in your backup strategy
5. **Set appropriate file permissions** (0o755 for directories, 0o644 for files)
6. **Use project-relative paths** for containerized deployments

### Common Storage Issues

**"ChromaDB permission denied" errors:**

```bash
# Fix permissions
chmod -R 755 ~/.local/share/CrewAI/
```

**"Database is locked" errors:**

```python
# Ensure only one CrewAI instance accesses storage
import fcntl
import os

storage_path = db_storage_path()
lock_file = os.path.join(storage_path, ".crewai.lock")

with open(lock_file, 'w') as f:
    fcntl.flock(f.fileno(), fcntl.LOCK_EX | fcntl.LOCK_NB)
    # Your CrewAI code here
```

**Storage not persisting between runs:**

```python
# Verify storage location is consistent
import os
print("CREWAI_STORAGE_DIR:", os.getenv("CREWAI_STORAGE_DIR"))
print("Current working directory:", os.getcwd())
print("Computed storage path:", db_storage_path())
```

## Custom Embedder Configuration

CrewAI supports multiple embedding providers to give you flexibility in choosing the best option for your use case. Here's a comprehensive guide to configuring different embedding providers for your memory system.

### Why Choose Different Embedding Providers?

* **Cost Optimization**: Local embeddings (Ollama) are free after initial setup
* **Privacy**: Keep your data local with Ollama or use your preferred cloud provider
* **Performance**: Some models work better for specific domains or languages
* **Consistency**: Match your embedding provider with your LLM provider
* **Compliance**: Meet specific regulatory or organizational requirements

### OpenAI Embeddings (Default)

OpenAI provides reliable, high-quality embeddings that work well for most use cases.

```python
from crewai import Crew

# Basic OpenAI configuration (uses environment OPENAI_API_KEY)
crew = Crew(
    agents=[...],
    tasks=[...],
    memory=True,
    embedder={
        "provider": "openai",
        "config": {
            "model": "text-embedding-3-small"  # or "text-embedding-3-large"
        }
    }
)

# Advanced OpenAI configuration
crew = Crew(
    memory=True,
    embedder={
        "provider": "openai",
        "config": {
            "api_key": "your-openai-api-key",  # Optional: override env var
            "model": "text-embedding-3-large",
            "dimensions": 1536,  # Optional: reduce dimensions for smaller storage
            "organization_id": "your-org-id"  # Optional: for organization accounts
        }
    }
)
```

### Azure OpenAI Embeddings

For enterprise users with Azure OpenAI deployments.

```python
crew = Crew(
    memory=True,
    embedder={
        "provider": "openai",  # Use openai provider for Azure
        "config": {
            "api_key": "your-azure-api-key",
            "api_base": "https://your-resource.openai.azure.com/",
            "api_type": "azure",
            "api_version": "2023-05-15",
            "model": "text-embedding-3-small",
            "deployment_id": "your-deployment-name"  # Azure deployment name
        }
    }
)
```

### Google AI Embeddings

Use Google's text embedding models for integration with Google Cloud services.

```python
crew = Crew(
    memory=True,
    embedder={
        "provider": "google",
        "config": {
            "api_key": "your-google-api-key",
            "model": "text-embedding-004"  # or "text-embedding-preview-0409"
        }
    }
)
```

### Vertex AI Embeddings

For Google Cloud users with Vertex AI access.

```python
crew = Crew(
    memory=True,
    embedder={
        "provider": "vertexai",
        "config": {
            "project_id": "your-gcp-project-id",
            "region": "us-central1",  # or your preferred region
            "api_key": "your-service-account-key",
            "model_name": "textembedding-gecko"
        }
    }
)
```

### Ollama Embeddings (Local)

Run embeddings locally for privacy and cost savings.

```python
# First, install and run Ollama locally, then pull an embedding model:
# ollama pull mxbai-embed-large

crew = Crew(
    memory=True,
    embedder={
        "provider": "ollama",
        "config": {
            "model": "mxbai-embed-large",  # or "nomic-embed-text"
            "url": "http://localhost:11434/api/embeddings"  # Default Ollama URL
        }
    }
)

# For custom Ollama installations
crew = Crew(
    memory=True,
    embedder={
        "provider": "ollama",
        "config": {
            "model": "mxbai-embed-large",
            "url": "http://your-ollama-server:11434/api/embeddings"
        }
    }
)
```

### Cohere Embeddings

Use Cohere's embedding models for multilingual support.

```python
crew = Crew(
    memory=True,
    embedder={
        "provider": "cohere",
        "config": {
            "api_key": "your-cohere-api-key",
            "model": "embed-english-v3.0"  # or "embed-multilingual-v3.0"
        }
    }
)
```

### VoyageAI Embeddings

High-performance embeddings optimized for retrieval tasks.

```python
crew = Crew(
    memory=True,
    embedder={
        "provider": "voyageai",
        "config": {
            "api_key": "your-voyage-api-key",
            "model": "voyage-large-2",  # or "voyage-code-2" for code
            "input_type": "document"  # or "query"
        }
    }
)
```

### AWS Bedrock Embeddings

For AWS users with Bedrock access.

```python
crew = Crew(
    memory=True,
    embedder={
        "provider": "bedrock",
        "config": {
            "aws_access_key_id": "your-access-key",
            "aws_secret_access_key": "your-secret-key",
            "region_name": "us-east-1",
            "model": "amazon.titan-embed-text-v1"
        }
    }
)
```

### Hugging Face Embeddings

Use open-source models from Hugging Face.

```python
crew = Crew(
    memory=True,
    embedder={
        "provider": "huggingface",
        "config": {
            "api_key": "your-hf-token",  # Optional for public models
            "model": "sentence-transformers/all-MiniLM-L6-v2",
            "api_url": "https://api-inference.huggingface.co"  # or your custom endpoint
        }
    }
)
```

### IBM Watson Embeddings

For IBM Cloud users.

```python
crew = Crew(
    memory=True,
    embedder={
        "provider": "watson",
        "config": {
            "api_key": "your-watson-api-key",
            "url": "your-watson-instance-url",
            "model": "ibm/slate-125m-english-rtrvr"
        }
    }
)
```

### Choosing the Right Embedding Provider

| Provider         | Best For                 | Pros                      | Cons                    |
| :--------------- | :----------------------- | :------------------------ | :---------------------- |
| **OpenAI**       | General use, reliability | High quality, well-tested | Cost, requires API key  |
| **Ollama**       | Privacy, cost savings    | Free, local, private      | Requires local setup    |
| **Google AI**    | Google ecosystem         | Good performance          | Requires Google account |
| **Azure OpenAI** | Enterprise, compliance   | Enterprise features       | Complex setup           |
| **Cohere**       | Multilingual content     | Great language support    | Specialized use case    |
| **VoyageAI**     | Retrieval tasks          | Optimized for search      | Newer provider          |

### Environment Variable Configuration

For security, store API keys in environment variables:

```python
import os

# Set environment variables
os.environ["OPENAI_API_KEY"] = "your-openai-key"
os.environ["GOOGLE_API_KEY"] = "your-google-key"
os.environ["COHERE_API_KEY"] = "your-cohere-key"

# Use without exposing keys in code
crew = Crew(
    memory=True,
    embedder={
        "provider": "openai",
        "config": {
            "model": "text-embedding-3-small"
            # API key automatically loaded from environment
        }
    }
)
```

### Testing Different Embedding Providers

Compare embedding providers for your specific use case:

```python
from crewai import Crew
from crewai.utilities.paths import db_storage_path

# Test different providers with the same data
providers_to_test = [
    {
        "name": "OpenAI",
        "config": {
            "provider": "openai",
            "config": {"model": "text-embedding-3-small"}
        }
    },
    {
        "name": "Ollama",
        "config": {
            "provider": "ollama",
            "config": {"model": "mxbai-embed-large"}
        }
    }
]

for provider in providers_to_test:
    print(f"\nTesting {provider['name']} embeddings...")

    # Create crew with specific embedder
    crew = Crew(
        agents=[...],
        tasks=[...],
        memory=True,
        embedder=provider['config']
    )

    # Run your test and measure performance
    result = crew.kickoff()
    print(f"{provider['name']} completed successfully")
```

### Troubleshooting Embedding Issues

**Model not found errors:**

```python
# Verify model availability
from crewai.rag.embeddings.configurator import EmbeddingConfigurator

configurator = EmbeddingConfigurator()
try:
    embedder = configurator.configure_embedder({
        "provider": "ollama",
        "config": {"model": "mxbai-embed-large"}
    })
    print("Embedder configured successfully")
except Exception as e:
    print(f"Configuration error: {e}")
```

**API key issues:**

```python
import os

# Check if API keys are set
required_keys = ["OPENAI_API_KEY", "GOOGLE_API_KEY", "COHERE_API_KEY"]
for key in required_keys:
    if os.getenv(key):
        print(f"✅ {key} is set")
    else:
        print(f"❌ {key} is not set")
```

**Performance comparison:**

```python
import time

def test_embedding_performance(embedder_config, test_text="This is a test document"):
    start_time = time.time()

    crew = Crew(
        agents=[...],
        tasks=[...],
        memory=True,
        embedder=embedder_config
    )

    # Simulate memory operation
    crew.kickoff()

    end_time = time.time()
    return end_time - start_time

# Compare performance
openai_time = test_embedding_performance({
    "provider": "openai",
    "config": {"model": "text-embedding-3-small"}
})

ollama_time = test_embedding_performance({
    "provider": "ollama",
    "config": {"model": "mxbai-embed-large"}
})

print(f"OpenAI: {openai_time:.2f}s")
print(f"Ollama: {ollama_time:.2f}s")
```

## 2. User Memory with Mem0 (Legacy)

<Warning>
  **Legacy Approach**: While fully functional, this approach is considered legacy. For new projects requiring user-specific memory, consider using External Memory instead.
</Warning>

User Memory integrates with [Mem0](https://mem0.ai/) to provide user-specific memory that persists across sessions and integrates with the crew's contextual memory system.

### Prerequisites

```bash
pip install mem0ai
```

### Mem0 Cloud Configuration

```python
import os
from crewai import Crew, Process

# Set your Mem0 API key
os.environ["MEM0_API_KEY"] = "m0-your-api-key"

crew = Crew(
    agents=[...],
    tasks=[...],
    memory=True,  # Required for contextual memory integration
    memory_config={
        "provider": "mem0",
        "config": {"user_id": "john"},
        "user_memory": {}  # DEPRECATED: Will be removed in version 0.156.0 or on 2025-08-04, use external_memory instead
    },
    process=Process.sequential,
    verbose=True
)
```

### Advanced Mem0 Configuration

When using Mem0 Client, you can customize the memory configuration further, by using parameters like 'includes', 'excludes', 'custom\_categories', 'infer' and 'run\_id' (this is only for short-term memory).
You can find more details in the [Mem0 documentation](https://docs.mem0.ai/).

```python

new_categories = [
    {"lifestyle_management_concerns": "Tracks daily routines, habits, hobbies and interests including cooking, time management and work-life balance"},
    {"seeking_structure": "Documents goals around creating routines, schedules, and organized systems in various life areas"},
    {"personal_information": "Basic information about the user including name, preferences, and personality traits"}
]

crew = Crew(
    agents=[...],
    tasks=[...],
    memory=True,
    memory_config={
        "provider": "mem0",
        "config": {
            "user_id": "john",
            "org_id": "my_org_id",        # Optional
            "project_id": "my_project_id", # Optional
            "api_key": "custom-api-key"    # Optional - overrides env var
            "run_id": "my_run_id",        # Optional - for short-term memory
            "includes": "include1",       # Optional 
            "excludes": "exclude1",       # Optional
            "infer": True                 # Optional defaults to True
            "custom_categories": new_categories  # Optional - custom categories for user memory
        },
        "user_memory": {}
    }
)
```

### Local Mem0 Configuration

```python
crew = Crew(
    agents=[...],
    tasks=[...],
    memory=True,
    memory_config={
        "provider": "mem0",
        "config": {
            "user_id": "john",
            "local_mem0_config": {
                "vector_store": {
                    "provider": "qdrant",
                    "config": {"host": "localhost", "port": 6333}
                },
                "llm": {
                    "provider": "openai",
                    "config": {"api_key": "your-api-key", "model": "gpt-4"}
                },
                "embedder": {
                    "provider": "openai",
                    "config": {"api_key": "your-api-key", "model": "text-embedding-3-small"}
                }
            },
            "infer": True                   # Optional defaults to True
        },
        "user_memory": {}
    }
)
```

## 3. External Memory (New Approach)

External Memory provides a standalone memory system that operates independently from the crew's built-in memory. This is ideal for specialized memory providers or cross-application memory sharing.

### Basic External Memory with Mem0

```python
import os
from crewai import Agent, Crew, Process, Task
from crewai.memory.external.external_memory import ExternalMemory

os.environ["MEM0_API_KEY"] = "your-api-key"

# Create external memory instance
external_memory = ExternalMemory(
    embedder_config={
        "provider": "mem0",
        "config": {"user_id": "U-123"}
    }
)

crew = Crew(
    agents=[...],
    tasks=[...],
    external_memory=external_memory,  # Separate from basic memory
    process=Process.sequential,
    verbose=True
)
```

### Custom Storage Implementation

```python
from crewai.memory.external.external_memory import ExternalMemory
from crewai.memory.storage.interface import Storage

class CustomStorage(Storage):
    def __init__(self):
        self.memories = []

    def save(self, value, metadata=None, agent=None):
        self.memories.append({
            "value": value,
            "metadata": metadata,
            "agent": agent
        })

    def search(self, query, limit=10, score_threshold=0.5):
        # Implement your search logic here
        return [m for m in self.memories if query.lower() in str(m["value"]).lower()]

    def reset(self):
        self.memories = []

# Use custom storage
external_memory = ExternalMemory(storage=CustomStorage())

crew = Crew(
    agents=[...],
    tasks=[...],
    external_memory=external_memory
)
```

## Memory System Comparison

| Feature              | Basic Memory        | User Memory (Legacy)       | External Memory   |
| -------------------- | ------------------- | -------------------------- | ----------------- |
| **Setup Complexity** | Simple              | Medium                     | Medium            |
| **Integration**      | Built-in contextual | Contextual + User-specific | Standalone        |
| **Storage**          | Local files         | Mem0 Cloud/Local           | Custom/Mem0       |
| **Cross-session**    | ✅                   | ✅                          | ✅                 |
| **User-specific**    | ❌                   | ✅                          | ✅                 |
| **Custom providers** | Limited             | Mem0 only                  | Any provider      |
| **Recommended for**  | Most use cases      | Legacy projects            | Specialized needs |

## Supported Embedding Providers

### OpenAI (Default)

```python
crew = Crew(
    memory=True,
    embedder={
        "provider": "openai",
        "config": {"model": "text-embedding-3-small"}
    }
)
```

### Ollama

```python
crew = Crew(
    memory=True,
    embedder={
        "provider": "ollama",
        "config": {"model": "mxbai-embed-large"}
    }
)
```

### Google AI

```python
crew = Crew(
    memory=True,
    embedder={
        "provider": "google",
        "config": {
            "api_key": "your-api-key",
            "model": "text-embedding-004"
        }
    }
)
```

### Azure OpenAI

```python
crew = Crew(
    memory=True,
    embedder={
        "provider": "openai",
        "config": {
            "api_key": "your-api-key",
            "api_base": "https://your-resource.openai.azure.com/",
            "api_version": "2023-05-15",
            "model_name": "text-embedding-3-small"
        }
    }
)
```

### Vertex AI

```python
crew = Crew(
    memory=True,
    embedder={
        "provider": "vertexai",
        "config": {
            "project_id": "your-project-id",
            "region": "your-region",
            "api_key": "your-api-key",
            "model_name": "textembedding-gecko"
        }
    }
)
```

## Security Best Practices

### Environment Variables

```python
import os
from crewai import Crew

# Store sensitive data in environment variables
crew = Crew(
    memory=True,
    embedder={
        "provider": "openai",
        "config": {
            "api_key": os.getenv("OPENAI_API_KEY"),
            "model": "text-embedding-3-small"
        }
    }
)
```

### Storage Security

```python
import os
from crewai import Crew
from crewai.memory import LongTermMemory
from crewai.memory.storage.ltm_sqlite_storage import LTMSQLiteStorage

# Use secure storage paths
storage_path = os.getenv("CREWAI_STORAGE_DIR", "./storage")
os.makedirs(storage_path, mode=0o700, exist_ok=True)  # Restricted permissions

crew = Crew(
    memory=True,
    long_term_memory=LongTermMemory(
        storage=LTMSQLiteStorage(
            db_path=f"{storage_path}/memory.db"
        )
    )
)
```

## Troubleshooting

### Common Issues

**Memory not persisting between sessions?**

* Check `CREWAI_STORAGE_DIR` environment variable
* Ensure write permissions to storage directory
* Verify memory is enabled with `memory=True`

**Mem0 authentication errors?**

* Verify `MEM0_API_KEY` environment variable is set
* Check API key permissions on Mem0 dashboard
* Ensure `mem0ai` package is installed

**High memory usage with large datasets?**

* Consider using External Memory with custom storage
* Implement pagination in custom storage search methods
* Use smaller embedding models for reduced memory footprint

### Performance Tips

* Use `memory=True` for most use cases (simplest and fastest)
* Only use User Memory if you need user-specific persistence
* Consider External Memory for high-scale or specialized requirements
* Choose smaller embedding models for faster processing
* Set appropriate search limits to control memory retrieval size

## Benefits of Using CrewAI's Memory System

* 🦾 **Adaptive Learning:** Crews become more efficient over time, adapting to new information and refining their approach to tasks.
* 🫡 **Enhanced Personalization:** Memory enables agents to remember user preferences and historical interactions, leading to personalized experiences.
* 🧠 **Improved Problem Solving:** Access to a rich memory store aids agents in making more informed decisions, drawing on past learnings and contextual insights.

## Memory Events

CrewAI's event system provides powerful insights into memory operations. By leveraging memory events, you can monitor, debug, and optimize your memory system's performance and behavior.

### Available Memory Events

CrewAI emits the following memory-related events:

| Event                             | Description                                                 | Key Properties                                                  |
| :-------------------------------- | :---------------------------------------------------------- | :-------------------------------------------------------------- |
| **MemoryQueryStartedEvent**       | Emitted when a memory query begins                          | `query`, `limit`, `score_threshold`                             |
| **MemoryQueryCompletedEvent**     | Emitted when a memory query completes successfully          | `query`, `results`, `limit`, `score_threshold`, `query_time_ms` |
| **MemoryQueryFailedEvent**        | Emitted when a memory query fails                           | `query`, `limit`, `score_threshold`, `error`                    |
| **MemorySaveStartedEvent**        | Emitted when a memory save operation begins                 | `value`, `metadata`, `agent_role`                               |
| **MemorySaveCompletedEvent**      | Emitted when a memory save operation completes successfully | `value`, `metadata`, `agent_role`, `save_time_ms`               |
| **MemorySaveFailedEvent**         | Emitted when a memory save operation fails                  | `value`, `metadata`, `agent_role`, `error`                      |
| **MemoryRetrievalStartedEvent**   | Emitted when memory retrieval for a task prompt starts      | `task_id`                                                       |
| **MemoryRetrievalCompletedEvent** | Emitted when memory retrieval completes successfully        | `task_id`, `memory_content`, `retrieval_time_ms`                |

### Practical Applications

#### 1. Memory Performance Monitoring

Track memory operation timing to optimize your application:

```python
from crewai.utilities.events.base_event_listener import BaseEventListener
from crewai.utilities.events import (
    MemoryQueryCompletedEvent,
    MemorySaveCompletedEvent
)
import time

class MemoryPerformanceMonitor(BaseEventListener):
    def __init__(self):
        super().__init__()
        self.query_times = []
        self.save_times = []

    def setup_listeners(self, crewai_event_bus):
        @crewai_event_bus.on(MemoryQueryCompletedEvent)
        def on_memory_query_completed(source, event: MemoryQueryCompletedEvent):
            self.query_times.append(event.query_time_ms)
            print(f"Memory query completed in {event.query_time_ms:.2f}ms. Query: '{event.query}'")
            print(f"Average query time: {sum(self.query_times)/len(self.query_times):.2f}ms")

        @crewai_event_bus.on(MemorySaveCompletedEvent)
        def on_memory_save_completed(source, event: MemorySaveCompletedEvent):
            self.save_times.append(event.save_time_ms)
            print(f"Memory save completed in {event.save_time_ms:.2f}ms")
            print(f"Average save time: {sum(self.save_times)/len(self.save_times):.2f}ms")

# Create an instance of your listener
memory_monitor = MemoryPerformanceMonitor()
```

#### 2. Memory Content Logging

Log memory operations for debugging and insights:

```python
from crewai.utilities.events.base_event_listener import BaseEventListener
from crewai.utilities.events import (
    MemorySaveStartedEvent,
    MemoryQueryStartedEvent,
    MemoryRetrievalCompletedEvent
)
import logging

# Configure logging
logger = logging.getLogger('memory_events')

class MemoryLogger(BaseEventListener):
    def setup_listeners(self, crewai_event_bus):
        @crewai_event_bus.on(MemorySaveStartedEvent)
        def on_memory_save_started(source, event: MemorySaveStartedEvent):
            if event.agent_role:
                logger.info(f"Agent '{event.agent_role}' saving memory: {event.value[:50]}...")
            else:
                logger.info(f"Saving memory: {event.value[:50]}...")

        @crewai_event_bus.on(MemoryQueryStartedEvent)
        def on_memory_query_started(source, event: MemoryQueryStartedEvent):
            logger.info(f"Memory query started: '{event.query}' (limit: {event.limit})")

        @crewai_event_bus.on(MemoryRetrievalCompletedEvent)
        def on_memory_retrieval_completed(source, event: MemoryRetrievalCompletedEvent):
            if event.task_id:
                logger.info(f"Memory retrieved for task {event.task_id} in {event.retrieval_time_ms:.2f}ms")
            else:
                logger.info(f"Memory retrieved in {event.retrieval_time_ms:.2f}ms")
            logger.debug(f"Memory content: {event.memory_content}")

# Create an instance of your listener
memory_logger = MemoryLogger()
```

#### 3. Error Tracking and Notifications

Capture and respond to memory errors:

```python
from crewai.utilities.events.base_event_listener import BaseEventListener
from crewai.utilities.events import (
    MemorySaveFailedEvent,
    MemoryQueryFailedEvent
)
import logging
from typing import Optional

# Configure logging
logger = logging.getLogger('memory_errors')

class MemoryErrorTracker(BaseEventListener):
    def __init__(self, notify_email: Optional[str] = None):
        super().__init__()
        self.notify_email = notify_email
        self.error_count = 0

    def setup_listeners(self, crewai_event_bus):
        @crewai_event_bus.on(MemorySaveFailedEvent)
        def on_memory_save_failed(source, event: MemorySaveFailedEvent):
            self.error_count += 1
            agent_info = f"Agent '{event.agent_role}'" if event.agent_role else "Unknown agent"
            error_message = f"Memory save failed: {event.error}. {agent_info}"
            logger.error(error_message)

            if self.notify_email and self.error_count % 5 == 0:
                self._send_notification(error_message)

        @crewai_event_bus.on(MemoryQueryFailedEvent)
        def on_memory_query_failed(source, event: MemoryQueryFailedEvent):
            self.error_count += 1
            error_message = f"Memory query failed: {event.error}. Query: '{event.query}'"
            logger.error(error_message)

            if self.notify_email and self.error_count % 5 == 0:
                self._send_notification(error_message)

    def _send_notification(self, message):
        # Implement your notification system (email, Slack, etc.)
        print(f"[NOTIFICATION] Would send to {self.notify_email}: {message}")

# Create an instance of your listener
error_tracker = MemoryErrorTracker(notify_email="admin@example.com")
```

### Integrating with Analytics Platforms

Memory events can be forwarded to analytics and monitoring platforms to track performance metrics, detect anomalies, and visualize memory usage patterns:

```python
from crewai.utilities.events.base_event_listener import BaseEventListener
from crewai.utilities.events import (
    MemoryQueryCompletedEvent,
    MemorySaveCompletedEvent
)

class MemoryAnalyticsForwarder(BaseEventListener):
    def __init__(self, analytics_client):
        super().__init__()
        self.client = analytics_client

    def setup_listeners(self, crewai_event_bus):
        @crewai_event_bus.on(MemoryQueryCompletedEvent)
        def on_memory_query_completed(source, event: MemoryQueryCompletedEvent):
            # Forward query metrics to analytics platform
            self.client.track_metric({
                "event_type": "memory_query",
                "query": event.query,
                "duration_ms": event.query_time_ms,
                "result_count": len(event.results) if hasattr(event.results, "__len__") else 0,
                "timestamp": event.timestamp
            })

        @crewai_event_bus.on(MemorySaveCompletedEvent)
        def on_memory_save_completed(source, event: MemorySaveCompletedEvent):
            # Forward save metrics to analytics platform
            self.client.track_metric({
                "event_type": "memory_save",
                "agent_role": event.agent_role,
                "duration_ms": event.save_time_ms,
                "timestamp": event.timestamp
            })
```

### Best Practices for Memory Event Listeners

1. **Keep handlers lightweight**: Avoid complex processing in event handlers to prevent performance impacts
2. **Use appropriate logging levels**: Use INFO for normal operations, DEBUG for details, ERROR for issues
3. **Batch metrics when possible**: Accumulate metrics before sending to external systems
4. **Handle exceptions gracefully**: Ensure your event handlers don't crash due to unexpected data
5. **Consider memory consumption**: Be mindful of storing large amounts of event data

## Conclusion

Integrating CrewAI's memory system into your projects is straightforward. By leveraging the provided memory components and configurations,
you can quickly empower your agents with the ability to remember, reason, and learn from their interactions, unlocking new levels of intelligence and capability.
