# Quickstart

> Build your first AI agent with CrewAI in under 5 minutes.

## Build your first CrewAI Agent

Let's create a simple crew that will help us `research` and `report` on the `latest AI developments` for a given topic or subject.

Before we proceed, make sure you have finished installing CrewAI.
If you haven't installed them yet, you can do so by following the [installation guide](/en/installation).

Follow the steps below to get Crewing! 🚣‍♂️

<Steps>
  <Step title="Create your crew">
    Create a new crew project by running the following command in your terminal.
    This will create a new directory called `latest-ai-development` with the basic structure for your crew.

    <CodeGroup>
      ```shell Terminal
      crewai create crew latest-ai-development
      ```
    </CodeGroup>
  </Step>

  <Step title="Navigate to your new crew project">
    <CodeGroup>
      ```shell Terminal
      cd latest-ai-development
      ```
    </CodeGroup>
  </Step>

  <Step title="Modify your `agents.yaml` file">
    <Tip>
      You can also modify the agents as needed to fit your use case or copy and paste as is to your project.
      Any variable interpolated in your `agents.yaml` and `tasks.yaml` files like `{topic}` will be replaced by the value of the variable in the `main.py` file.
    </Tip>

    ```yaml agents.yaml
    # src/latest_ai_development/config/agents.yaml
    researcher:
      role: >
        {topic} Senior Data Researcher
      goal: >
        Uncover cutting-edge developments in {topic}
      backstory: >
        You're a seasoned researcher with a knack for uncovering the latest
        developments in {topic}. Known for your ability to find the most relevant
        information and present it in a clear and concise manner.

    reporting_analyst:
      role: >
        {topic} Reporting Analyst
      goal: >
        Create detailed reports based on {topic} data analysis and research findings
      backstory: >
        You're a meticulous analyst with a keen eye for detail. You're known for
        your ability to turn complex data into clear and concise reports, making
        it easy for others to understand and act on the information you provide.
    ```
  </Step>

  <Step title="Modify your `tasks.yaml` file">
    ````yaml tasks.yaml
    # src/latest_ai_development/config/tasks.yaml
    research_task:
      description: >
        Conduct a thorough research about {topic}
        Make sure you find any interesting and relevant information given
        the current year is 2025.
      expected_output: >
        A list with 10 bullet points of the most relevant information about {topic}
      agent: researcher

    reporting_task:
      description: >
        Review the context you got and expand each topic into a full section for a report.
        Make sure the report is detailed and contains any and all relevant information.
      expected_output: >
        A fully fledge reports with the mains topics, each with a full section of information.
        Formatted as markdown without '```'
      agent: reporting_analyst
      output_file: report.md
    ````
  </Step>

  <Step title="Modify your `crew.py` file">
    ```python crew.py
    # src/latest_ai_development/crew.py
    from crewai import Agent, Crew, Process, Task
    from crewai.project import CrewBase, agent, crew, task
    from crewai_tools import SerperDevTool
    from crewai.agents.agent_builder.base_agent import BaseAgent
    from typing import List

    @CrewBase
    class LatestAiDevelopmentCrew():
      """LatestAiDevelopment crew"""

      agents: List[BaseAgent]
      tasks: List[Task]

      @agent
      def researcher(self) -> Agent:
        return Agent(
          config=self.agents_config['researcher'], # type: ignore[index]
          verbose=True,
          tools=[SerperDevTool()]
        )

      @agent
      def reporting_analyst(self) -> Agent:
        return Agent(
          config=self.agents_config['reporting_analyst'], # type: ignore[index]
          verbose=True
        )

      @task
      def research_task(self) -> Task:
        return Task(
          config=self.tasks_config['research_task'], # type: ignore[index]
        )

      @task
      def reporting_task(self) -> Task:
        return Task(
          config=self.tasks_config['reporting_task'], # type: ignore[index]
          output_file='output/report.md' # This is the file that will be contain the final report.
        )

      @crew
      def crew(self) -> Crew:
        """Creates the LatestAiDevelopment crew"""
        return Crew(
          agents=self.agents, # Automatically created by the @agent decorator
          tasks=self.tasks, # Automatically created by the @task decorator
          process=Process.sequential,
          verbose=True,
        )
    ```
  </Step>

  <Step title="[Optional] Add before and after crew functions">
    ```python crew.py
    # src/latest_ai_development/crew.py
    from crewai import Agent, Crew, Process, Task
    from crewai.project import CrewBase, agent, crew, task, before_kickoff, after_kickoff
    from crewai_tools import SerperDevTool

    @CrewBase
    class LatestAiDevelopmentCrew():
      """LatestAiDevelopment crew"""

      @before_kickoff
      def before_kickoff_function(self, inputs):
        print(f"Before kickoff function with inputs: {inputs}")
        return inputs # You can return the inputs or modify them as needed

      @after_kickoff
      def after_kickoff_function(self, result):
        print(f"After kickoff function with result: {result}")
        return result # You can return the result or modify it as needed

      # ... remaining code
    ```
  </Step>

  <Step title="Feel free to pass custom inputs to your crew">
    For example, you can pass the `topic` input to your crew to customize the research and reporting.

    ```python main.py
    #!/usr/bin/env python
    # src/latest_ai_development/main.py
    import sys
    from latest_ai_development.crew import LatestAiDevelopmentCrew

    def run():
      """
      Run the crew.
      """
      inputs = {
        'topic': 'AI Agents'
      }
      LatestAiDevelopmentCrew().crew().kickoff(inputs=inputs)
    ```
  </Step>

  <Step title="Set your environment variables">
    Before running your crew, make sure you have the following keys set as environment variables in your `.env` file:

    * A [Serper.dev](https://serper.dev/) API key: `SERPER_API_KEY=YOUR_KEY_HERE`
    * The configuration for your choice of model, such as an API key. See the
      [LLM setup guide](/en/concepts/llms#setting-up-your-llm) to learn how to configure models from any provider.
  </Step>

  <Step title="Lock and install the dependencies">
    * Lock the dependencies and install them by using the CLI command:
      <CodeGroup>
        ```shell Terminal
        crewai install
        ```
      </CodeGroup>
    * If you have additional packages that you want to install, you can do so by running:
      <CodeGroup>
        ```shell Terminal
        uv add <package-name>
        ```
      </CodeGroup>
  </Step>

  <Step title="Run your crew">
    * To run your crew, execute the following command in the root of your project:
      <CodeGroup>
        ```bash Terminal
        crewai run
        ```
      </CodeGroup>
  </Step>

  <Step title="Enterprise Alternative: Create in Crew Studio">
    For CrewAI Enterprise users, you can create the same crew without writing code:

    1. Log in to your CrewAI Enterprise account (create a free account at [app.crewai.com](https://app.crewai.com))
    2. Open Crew Studio
    3. Type what is the automation you're trying to build
    4. Create your tasks visually and connect them in sequence
    5. Configure your inputs and click "Download Code" or "Deploy"

    ![Crew Studio Quickstart](https://mintlify.s3.us-west-1.amazonaws.com/crewai/images/enterprise/crew-studio-interface.png)

    <Card title="Try CrewAI Enterprise" icon="rocket" href="https://app.crewai.com">
      Start your free account at CrewAI Enterprise
    </Card>
  </Step>

  <Step title="View your final report">
    You should see the output in the console and the `report.md` file should be created in the root of your project with the final report.

    Here's an example of what the report should look like:

    <CodeGroup>
      ```markdown output/report.md
      # Comprehensive Report on the Rise and Impact of AI Agents in 2025

      ## 1. Introduction to AI Agents
      In 2025, Artificial Intelligence (AI) agents are at the forefront of innovation across various industries. As intelligent systems that can perform tasks typically requiring human cognition, AI agents are paving the way for significant advancements in operational efficiency, decision-making, and overall productivity within sectors like Human Resources (HR) and Finance. This report aims to detail the rise of AI agents, their frameworks, applications, and potential implications on the workforce.

      ## 2. Benefits of AI Agents
      AI agents bring numerous advantages that are transforming traditional work environments. Key benefits include:

      - **Task Automation**: AI agents can carry out repetitive tasks such as data entry, scheduling, and payroll processing without human intervention, greatly reducing the time and resources spent on these activities.
      - **Improved Efficiency**: By quickly processing large datasets and performing analyses that would take humans significantly longer, AI agents enhance operational efficiency. This allows teams to focus on strategic tasks that require higher-level thinking.
      - **Enhanced Decision-Making**: AI agents can analyze trends and patterns in data, provide insights, and even suggest actions, helping stakeholders make informed decisions based on factual data rather than intuition alone.

      ## 3. Popular AI Agent Frameworks
      Several frameworks have emerged to facilitate the development of AI agents, each with its own unique features and capabilities. Some of the most popular frameworks include:

      - **Autogen**: A framework designed to streamline the development of AI agents through automation of code generation.
      - **Semantic Kernel**: Focuses on natural language processing and understanding, enabling agents to comprehend user intentions better.
      - **Promptflow**: Provides tools for developers to create conversational agents that can navigate complex interactions seamlessly.
      - **Langchain**: Specializes in leveraging various APIs to ensure agents can access and utilize external data effectively.
      - **CrewAI**: Aimed at collaborative environments, CrewAI strengthens teamwork by facilitating communication through AI-driven insights.
      - **MemGPT**: Combines memory-optimized architectures with generative capabilities, allowing for more personalized interactions with users.

      These frameworks empower developers to build versatile and intelligent agents that can engage users, perform advanced analytics, and execute various tasks aligned with organizational goals.

      ## 4. AI Agents in Human Resources
      AI agents are revolutionizing HR practices by automating and optimizing key functions:

      - **Recruiting**: AI agents can screen resumes, schedule interviews, and even conduct initial assessments, thus accelerating the hiring process while minimizing biases.
      - **Succession Planning**: AI systems analyze employee performance data and potential, helping organizations identify future leaders and plan appropriate training.
      - **Employee Engagement**: Chatbots powered by AI can facilitate feedback loops between employees and management, promoting an open culture and addressing concerns promptly.

      As AI continues to evolve, HR departments leveraging these agents can realize substantial improvements in both efficiency and employee satisfaction.

      ## 5. AI Agents in Finance
      The finance sector is seeing extensive integration of AI agents that enhance financial practices:

      - **Expense Tracking**: Automated systems manage and monitor expenses, flagging anomalies and offering recommendations based on spending patterns.
      - **Risk Assessment**: AI models assess credit risk and uncover potential fraud by analyzing transaction data and behavioral patterns.
      - **Investment Decisions**: AI agents provide stock predictions and analytics based on historical data and current market conditions, empowering investors with informative insights.

      The incorporation of AI agents into finance is fostering a more responsive and risk-aware financial landscape.

      ## 6. Market Trends and Investments
      The growth of AI agents has attracted significant investment, especially amidst the rising popularity of chatbots and generative AI technologies. Companies and entrepreneurs are eager to explore the potential of these systems, recognizing their ability to streamline operations and improve customer engagement.

      Conversely, corporations like Microsoft are taking strides to integrate AI agents into their product offerings, with enhancements to their Copilot 365 applications. This strategic move emphasizes the importance of AI literacy in the modern workplace and indicates the stabilizing of AI agents as essential business tools.

      ## 7. Future Predictions and Implications
      Experts predict that AI agents will transform essential aspects of work life. As we look toward the future, several anticipated changes include:

      - Enhanced integration of AI agents across all business functions, creating interconnected systems that leverage data from various departmental silos for comprehensive decision-making.
      - Continued advancement of AI technologies, resulting in smarter, more adaptable agents capable of learning and evolving from user interactions.
      - Increased regulatory scrutiny to ensure ethical use, especially concerning data privacy and employee surveillance as AI agents become more prevalent.

      To stay competitive and harness the full potential of AI agents, organizations must remain vigilant about latest developments in AI technology and consider continuous learning and adaptation in their strategic planning.

      ## 8. Conclusion
      The emergence of AI agents is undeniably reshaping the workplace landscape in 5. With their ability to automate tasks, enhance efficiency, and improve decision-making, AI agents are critical in driving operational success. Organizations must embrace and adapt to AI developments to thrive in an increasingly digital business environment.
      ```
    </CodeGroup>
  </Step>
</Steps>

<Check>
  Congratulations!

  You have successfully set up your crew project and are ready to start building your own agentic workflows!
</Check>

### Note on Consistency in Naming

The names you use in your YAML files (`agents.yaml` and `tasks.yaml`) should match the method names in your Python code.
For example, you can reference the agent for specific tasks from `tasks.yaml` file.
This naming consistency allows CrewAI to automatically link your configurations with your code; otherwise, your task won't recognize the reference properly.

#### Example References

<Tip>
  Note how we use the same name for the agent in the `agents.yaml` (`email_summarizer`) file as the method name in the `crew.py` (`email_summarizer`) file.
</Tip>

```yaml agents.yaml
email_summarizer:
    role: >
      Email Summarizer
    goal: >
      Summarize emails into a concise and clear summary
    backstory: >
      You will create a 5 bullet point summary of the report
    llm: provider/model-id  # Add your choice of model here
```

<Tip>
  Note how we use the same name for the task in the `tasks.yaml` (`email_summarizer_task`) file as the method name in the `crew.py` (`email_summarizer_task`) file.
</Tip>

```yaml tasks.yaml
email_summarizer_task:
    description: >
      Summarize the email into a 5 bullet point summary
    expected_output: >
      A 5 bullet point summary of the email
    agent: email_summarizer
    context:
      - reporting_task
      - research_task
```

## Deploying Your Crew

The easiest way to deploy your crew to production is through [CrewAI Enterprise](http://app.crewai.com).

Watch this video tutorial for a step-by-step demonstration of deploying your crew to [CrewAI Enterprise](http://app.crewai.com) using the CLI.

<iframe width="100%" height="400" src="https://www.youtube.com/embed/3EqSV-CYDZA" title="CrewAI Deployment Guide" frameborder="0" style={{ borderRadius: '10px' }} allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen />

<CardGroup cols={2}>
  <Card title="Deploy on Enterprise" icon="rocket" href="http://app.crewai.com">
    Get started with CrewAI Enterprise and deploy your crew in a production environment with just a few clicks.
  </Card>

  <Card title="Join the Community" icon="comments" href="https://community.crewai.com">
    Join our open source community to discuss ideas, share your projects, and connect with other CrewAI developers.
  </Card>
</CardGroup>
