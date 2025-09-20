---
layout: post
title: "Building Intelligent SQL Agents: A Baseline Implementation with LangGraph"
date: 2025-09-12 10:00:00 -0000
categories: ai sql agents langgraph database
tags: [AI, SQL, LangGraph, Database, Agent, NLP, LLM, Reasoning]
---

As large language models become increasingly sophisticated, one of their most promising applications is bridging the gap between natural language and database queries. In this post, I'll walk through my baseline implementation of an intelligent SQL agent using LangGraph that can understand natural language questions and translate them into accurate SQL queries. This is the first notebook in a series where I'll explore increasingly complex SQL agent architectures, with this implementation serving as our foundation for comparison.

## What is a SQL Agent?

A SQL agent represents a sophisticated bridge between human language and database systems. At its core, it transforms natural language questions into precise SQL queries, but the magic lies in how it accomplishes this transformation. The agent must first understand what you're really asking, then explore the database structure to find the relevant tables and relationships, construct syntactically correct queries that capture your intent, and finally translate the raw results back into meaningful answers.

The agent presented in this post demonstrates these capabilities using the Chinook sample database, a rich dataset representing a digital music store complete with artists, albums, tracks, customers, and sales transactions. This choice provides an excellent testing ground because it mirrors real-world business scenarios where users might ask everything from simple counts to complex analytical questions about customer behavior and sales trends. This implementation was largely inspired by an example from Langchain: https://python.langchain.com/docs/tutorials/sql_qa/

## Core Implementation Analysis

### Model Configuration and Safety

The agent uses a local Ollama model with specific safety and performance considerations:

```python
# Deterministic SQL generation for consistency
llm = ChatOllama(model="qwen3:32b", temperature=0, verbose=True)

# Comprehensive safety guidelines in the prompt
prompt_template = ChatPromptTemplate.from_template(
    "You are an agent designed to interact with a SQL database. "
    # Safety constraints to prevent destructive operations
    "DO NOT make any DML statements (INSERT, UPDATE, DELETE, DROP etc.) to the database. "
    # Performance optimization
    "Do not LIMIT your queries if you are doing aggregation. "
    "LIMIT the query if it is going to return more than 10000 rows."
)
```

Safety takes precedence through explicit constraints built directly into the prompt template. Rather than relying on the model's training to avoid destructive operations, the system explicitly prohibits any data modification commands, creating multiple layers of protection against accidental damage. The performance guidelines embedded in the prompt help the agent make smart optimization decisions, like knowing when to use LIMIT clauses and how to structure aggregation queries for maximum efficiency.

### Database Engine Setup

The implementation uses an in-memory SQLite approach with the Chinook sample database:

```python
def get_engine_for_chinook_db():
    """Download and set up Chinook database in memory for fast queries."""
    url = "https://raw.githubusercontent.com/lerocha/chinook-database/master/ChinookDatabase/DataSources/Chinook_Sqlite.sql"
    response = requests.get(url)
    connection = sqlite3.connect(":memory:", check_same_thread=False)
    connection.executescript(response.text)

    return create_engine(
        "sqlite://",
        creator=lambda: connection,
        poolclass=StaticPool,
        connect_args={"check_same_thread": False}
    )
```

By keeping everything in memory, the agent can execute even complex queries with minimal latency, eliminating the I/O bottlenecks that typically slow down database operations. 

Perhaps most valuable for development and demonstration purposes is the reproducibility factor. Every time the agent starts up, it works with exactly the same dataset, making it easy to test improvements and compare results across different versions. The self-contained nature eliminates the setup friction that often prevents people from experimenting with database agents – no need to install PostgreSQL, configure MySQL, or worry about connection strings and permissions.

## Architecture Overview

Rather than jumping straight into query generation, the agent follows a methodical approach that mirrors how a skilled database analyst would tackle an unfamiliar question. It begins by carefully analyzing what you're actually asking, often reading between the lines to understand implicit requirements. Next comes the detective work: systematically exploring the database structure to identify relevant tables and understand how they relate to each other.

Once the agent has mapped out the data landscape, it constructs SQL queries that capture not just the literal request but the underlying analytical intent. Before execution, it performs a critical validation step, double-checking syntax and logic to prevent errors. Finally, it executes the query and transforms the raw database output into a natural language response that directly answers your original question.

### Key Components

**LangGraph State Management**: The agent uses a user-defined state that extends MessagesState to track conversation flow and ensure consistent responses:

```python
class QueryResponse(BaseModel):
    """Structured response model for the agent's final output."""
    answer: str = Field(
        description="The answer to the user's question based on database query results"
    )

class AgentState(MessagesState):
    """State management for the SQL agent workflow.

    Extends MessagesState to track conversation history while adding
    a structured final response field for consistent output formatting.
    """
    final_response: QueryResponse
```

This architecture delievers powerful capabilities that elevate the agent beyond simple query generation. The Pydantic models provides type safety, ensuring that every response follows a consistent structure regardless of query complexity. Meanwhile, the MessagesState component maintains a complete memory of the conversation, allowing the agent to reference previous questions and build upon earlier discoveries. Perhaps most importantly, the structured output system guarantees that responses are always formatted predictably, making the agent suitable for integration into larger applications where consistent data formats are crucial.

**SQL Toolkit Integration**: The agent leverages LangChain's `SQLDatabaseToolkit` which provides four specialized tools for safe database interaction:

```python
# Initialize the SQL toolkit with database and language model
toolkit = SQLDatabaseToolkit(db=db, llm=llm)
tools = toolkit.get_tools()
```

Each tool in the toolkit serves as a specialized instrument for the agent. The `QuerySQLDatabaseTool` acts as the execution engine, running validated queries and returning properly formatted results. The `InfoSQLDatabaseTool` functions like a database explorer, diving deep into table structures to reveal column types, constraints, and sample data that help the agent understand what it's working with. The `ListSQLDatabaseTool` provides the initial reconnaissance, scanning the entire database to catalog available tables and give the agent its bearings.

Perhaps most critically, the `QuerySQLCheckerTool` serves as a safety net and quality assurance layer, scrutinizing every query before execution to catch syntax errors, logical mistakes, and potential performance issues. This toolkit approach creates a cohesive ecosystem where each tool has a clear role, and together they provide both the power and safety needed for autonomous database interaction.

**Workflow Orchestration**: The final agent is composed by a graph with three main nodes and conditional routing logic:

```python
# Define the agent workflow nodes
workflow.add_node("agent", call_model)        # SQL query generation and reasoning
workflow.add_node("respond", respond)         # Final response formatting
workflow.add_node("tools", ToolNode(tools))   # Tool execution (database operations)

# Conditional routing based on agent decisions
workflow.add_conditional_edges(
    "agent",
    should_continue,
    {
        "continue": "tools",    # Execute database tools
        "respond": "respond",   # Generate final response
    },
)

# Create feedback loop for iterative reasoning
workflow.add_edge("tools", "agent")
workflow.add_edge("respond", END)
```

What makes this architecture particularly appealing is how it enables dynamic, context-aware decision making throughout the query process. The `should_continue` function acts as the agent's internal router, constantly evaluating whether more database exploration is needed or if enough information has been gathered to provide a final answer. This creates a feedback loop where tool execution informs further reasoning, allowing the agent to adapt its approach based on what it discovers.

The system also incorporates safeguards against runaway processes through deterministic termination conditions. Rather than risking infinite exploration cycles, the agent recognizes when it has sufficient information to answer the question or when it has reached reasonable limits on conversation length, ensuring reliable operation even with complex queries.

## Agent Decision Logic Deep Dive

### Control Flow Analysis

The agent's decision-making process is governed by the `should_continue` function, which implements a simple control logic:

```python
def should_continue(state: AgentState):
    """Determine workflow progression based on agent state."""
    messages = state["messages"]
    last_message = messages[-1]

    # Check termination conditions
    if not last_message.tool_calls or len(messages) > 60:
        return "respond"  # Move to final response generation
    else:
        return "continue"  # Continue with tool execution
```

**Decision Criteria Analysis**:
- **Tool Call Detection**: Checks if the model requested database tools
- **Conversation Length Limits**: Prevents infinite loops with 60-message cap
- **Binary Routing**: Simple but effective continue/respond decision

### Model Chain Architecture

The implementation uses two distinct model chains for different phases:

```python
# Phase 1: SQL Generation with Tool Access
generator_model = llm.bind_tools(tools)
model_with_tools = generator_prompt | generator_model

# Phase 2: Structured Response Generation
model_with_structured_output = prompt | llm.with_structured_output(QueryResponse)
```

By separating query generation from response formatting, the system can optimize each phase independently. The generator model focuses entirely on understanding questions and crafting appropriate SQL, while the response model specializes in translating database results into clear, natural language answers.

The tool binding approach ensures that only the generator model has direct database access, creating a clean security boundary. Meanwhile, the structured output capability guarantees that no matter how complex the query or varied the results, the final response always follows a predictable format consumed by downstream applications.

## Technical Implementation Notes

The full implementation uses:
- **Qwen3:32b** as the language model (temperature=0 for consistency)
- **SQLite Chinook database** for realistic testing scenarios
- **LangGraph** for workflow orchestration and state management
- **Structured output** with Pydantic models for response consistency

Key safety features include:
- **No DML statements** (INSERT, UPDATE, DELETE, DROP prevention)
- **Query validation** before execution
- **Result limiting** to prevent overwhelming outputs
- **Schema exploration** to understand data before querying

## Demonstrating Multi-Step Reasoning

Let me show you how the agent handles different complexity levels of questions, demonstrating the code in action with detailed analysis of the execution flow.

### Simple Query: Artist Count
**Question**: "Total number of artists in the db?"

**Execution Trace Analysis**:
The agent follows the prescribed workflow with precision:

1. **Tool Discovery**: `sql_db_list_tables` → Returns 11 tables including "Artist"
2. **Schema Exploration**: `sql_db_schema` on "Artist" table → Reveals `ArtistId` (PK) and `Name` columns
3. **Query Construction**: Generates `SELECT COUNT(*) FROM Artist`
4. **Direct Execution**: Skips validation (simple query) → Returns `(275,)`
5. **Response Formatting**: Converts to natural language: "**275**"

This demonstrates the agent's efficiency in handling straightforward queries without unnecessary validation steps.

### Moderate Complexity: Album Popularity
**Question**: "Which albums are the most popular?"

**Complex Reasoning Analysis**:
This query demonstrates sophisticated multi-step reasoning and business logic interpretation:

1. **Semantic Interpretation**: Agent interprets "popular" as sales volume/frequency
2. **Schema Discovery**: Explores `Album`, `Track`, `InvoiceLine` tables to understand relationships
3. **Join Logic Construction**: Builds complex query joining three tables:
   ```sql
   SELECT a.Title, COUNT(il.Quantity) as purchase_count
   FROM Album a
   JOIN Track t ON a.AlbumId = t.AlbumId
   JOIN InvoiceLine il ON t.TrackId = il.TrackId
   GROUP BY a.AlbumId, a.Title
   ORDER BY purchase_count DESC
   ```
4. **Business Logic**: Correctly assumes sales data represents popularity

**Agent Reasoning Process**:
- **Domain Knowledge**: Understands that popularity in music correlates with purchase behavior
- **Relationship Mapping**: Automatically discovers Album → Track → InvoiceLine relationship chain
- **Aggregation Strategy**: Chooses COUNT over SUM for frequency-based popularity

This showcases the agent's ability to make contextually appropriate business assumptions.

### Complex Multi-Step Reasoning Challenge

Now comes the interesting part. I posed this complex business intelligence question:

**"What seasonal trends exist in music purchases, and how do they differ between digital vs physical media across different markets?"**

This question requires the agent to:
1. **Identify seasonal patterns** - extract temporal data from invoices
2. **Classify media types** - distinguish digital vs physical products
3. **Segment by markets** - analyze geographical differences
4. **Perform comparative analysis** - contrast trends across segments
5. **Synthesize insights** - combine multiple analytical dimensions

## Where the Baseline Agent Struggles

Here's where our current implementation hits its limitations, revealed through history trace:

### 1. **Linear Tool Execution Pattern**

```python
# Current implementation processes tools one at a time
def call_model(state: AgentState):
    response = model_with_tools.invoke(state["messages"])
    return {"messages": [response]}  # Single response per call
```

The agent's sequential approach, while methodical, reveals its limitations when faced with complex analytical questions that could benefit from parallel investigation. Rather than developing a comprehensive analysis strategy upfront, it tends to follow a linear path of discovery, executing one tool, processing the results, then deciding on the next step. This works well for straightforward queries but becomes inefficient when dealing with multi-faceted business intelligence questions that could be explored simultaneously across different dimensions.

### 2. **Schema Understanding Limitations**

The agent's approach to schema understanding, while functional for basic queries, reveals significant gaps when dealing with complex relational data. Although it can examine individual tables through the `sql_db_schema` tool, it lacks the ability to systematically map out foreign key relationships and understand how tables interconnect to form meaningful business entities.

This limitation becomes particularly apparent when users ask questions that require understanding of business concepts that span multiple tables. The agent might recognize that there are Customer, Order, and Product tables, but it struggles to grasp higher-level concepts like "customer lifetime value" or "product category performance" that require synthesizing data across these related entities. Without this business context awareness, the agent often produces technically correct but analytically shallow responses.

### 3. **Single-Pass Query Generation**

```python
# No iterative refinement in current workflow
workflow.add_edge("tools", "agent")  # Returns to agent but doesn't improve queries
```

The agent doesn't implement:
- **Query Result Analysis**: No evaluation of whether results actually answer the question
- **Iterative Improvement**: Cannot refine queries based on intermediate findings
- **Multi-Query Strategies**: Struggles with questions requiring multiple related queries

### 4. **Response Synthesis Gaps**

The structured output model is too simplistic:

```python
class QueryResponse(BaseModel):
    answer: str = Field(description="The answer based on database query results")
```

The current response structure, while clean and predictable, constrains the agent's ability to provide rich analytical insights. By limiting responses to a single string field, the system cannot effectively communicate multi-dimensional findings or provide the kind of structured analysis that business users often need.

Moreover, the absence of confidence scoring means users have no way to gauge how reliable the agent's answers might be. A query that returns results from a comprehensive dataset receives the same confident presentation as one based on limited or potentially questionable data. This lack of transparency about reasoning and methodology makes it difficult for users to understand how the agent arrived at its conclusions or to assess whether they should trust the results for important business decisions.

## Conclusion

This baseline SQL agent represents a solid foundation for natural language to SQL translation, successfully handling straightforward and moderately complex queries. However, the limitations exposed by complex multi-step reasoning questions highlight the need for more sophisticated architectures.

The journey from simple SQL translation to sophisticated reasoning is just beginning. Each limitation we've identified represents an opportunity to push the boundaries of what's possible with AI-powered database interaction.

## Next steps

With this baseline established my goal is to start adding more nodes to the graph in order to cover the deficiencies of the model.  


*Stay tuned for the next post in this series, where I'll implement a planning-enhanced SQL agent that can break down complex analytical questions into structured, executable analysis plans.*

---

**Notebook**: [View the complete implementation in the Jupyter notebook](https://github.com/bvcmartins/agentic_patterns/blob/main/src/sql/agent_sql_v1.ipynb)

**Repository**: The complete implementation is available in my [agentic_patterns repository](https://github.com/bvcmartins/agentic_patterns/tree/main/src/sql)
