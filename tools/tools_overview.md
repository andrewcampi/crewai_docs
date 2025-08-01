# Tools Overview

> Discover CrewAI's extensive library of 40+ tools to supercharge your AI agents

CrewAI provides an extensive library of pre-built tools to enhance your agents' capabilities. From file processing to web scraping, database queries to AI services - we've got you covered.

## **Tool Categories**

<CardGroup cols={2}>
  <Card title="File & Document" icon="file-check" href="/en/tools/file-document/overview" color="#3B82F6">
    Read, write, and search through various file formats including PDF, DOCX, JSON, CSV, and more. Perfect for document processing workflows.
  </Card>

  <Card title="Web Scraping & Browsing" icon="globe" href="/en/tools/web-scraping/overview" color="#10B981">
    Extract data from websites, automate browser interactions, and scrape content at scale with tools like Firecrawl, Selenium, and more.
  </Card>

  <Card title="Search & Research" icon="magnifying-glass" href="/en/tools/search-research/overview" color="#F59E0B">
    Perform web searches, find code repositories, research YouTube content, and discover information across the internet.
  </Card>

  <Card title="Database & Data" icon="database" href="/en/tools/database-data/overview" color="#8B5CF6">
    Connect to SQL databases, vector stores, and data warehouses. Query MySQL, PostgreSQL, Snowflake, Qdrant, and Weaviate.
  </Card>

  <Card title="AI & Machine Learning" icon="brain" href="/en/tools/ai-ml/overview" color="#EF4444">
    Generate images with DALL-E, process vision tasks, integrate with LangChain, build RAG systems, and leverage code interpreters.
  </Card>

  <Card title="Cloud & Storage" icon="cloud" href="/en/tools/cloud-storage/overview" color="#06B6D4">
    Interact with cloud services including AWS S3, Amazon Bedrock, and other cloud storage and AI services.
  </Card>

  <Card title="Automation & Integration" icon="bolt" href="/en/tools/automation/overview" color="#84CC16">
    Automate workflows with Apify, Composio, and other integration platforms to connect your agents with external services.
  </Card>
</CardGroup>

## **Quick Access**

Need a specific tool? Here are some popular choices:

<CardGroup cols={3}>
  <Card title="RAG Tool" icon="image" href="/en/tools/ai-ml/ragtool">
    Implement Retrieval-Augmented Generation
  </Card>

  <Card title="Serper Dev" icon="book-atlas" href="/en/tools/search-research/serperdevtool">
    Google search API
  </Card>

  <Card title="File Read" icon="file" href="/en/tools/file-document/filereadtool">
    Read any file type
  </Card>

  <Card title="Scrape Website" icon="globe" href="/en/tools/web-scraping/scrapewebsitetool">
    Extract web content
  </Card>

  <Card title="Code Interpreter" icon="code" href="/en/tools/ai-ml/codeinterpretertool">
    Execute Python code
  </Card>

  <Card title="S3 Reader" icon="cloud" href="/en/tools/cloud-storage/s3readertool">
    Access AWS S3 files
  </Card>
</CardGroup>

## **Getting Started**

To use any tool in your CrewAI project:

1. **Import** the tool in your crew configuration
2. **Add** it to your agent's tools list
3. **Configure** any required API keys or settings

```python
from crewai_tools import FileReadTool, SerperDevTool

# Add tools to your agent
agent = Agent(
    role="Research Analyst",
    tools=[FileReadTool(), SerperDevTool()],
    # ... other configuration
)
```

Ready to explore? Pick a category above to discover tools that fit your use case!
