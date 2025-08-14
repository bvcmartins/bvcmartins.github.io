---
layout: post
title: "Agent LATS: Enhancing AI Responses Through Tree Search"
date: 2025-08-12 12:00:00 -0000
categories: ai machine-learning tree-search reasoning
tags: [AI, Machine Learning, LATS, Tree Search, LLM, Reasoning, Agent, MCTS]
---

Language Agent Tree Search (LATS) is a fascinating approach to AI reasoning that combines language models with tree search algorithms to generate better responses through iterative refinement. In this post, I'll walk through my implementation of LATS and show how AI agents can systematically improve their outputs by exploring multiple solution paths.

## What is Agent LATS?

LATS (Language Agent Tree Search) enhances LLM responses by applying Monte Carlo Tree Search (MCTS) principles to language generation. The technique was introduced by Zhou et al. (2023) in their paper "[Language Agent Tree Search Unifies Reasoning Acting and Planning in Language Models](https://arxiv.org/abs/2310.04406)" (ICML 2024) as the first general framework that combines language model capabilities in reasoning, acting, and planning. Instead of producing a single response, LATS:

1. **Generates** an initial response
2. **Evaluates** its quality through structured reflection
3. **Expands** the search by creating alternative responses
4. **Selects** the most promising branches using Upper Confidence Bound (UCB)
5. **Iterates** until finding a satisfactory solution or reaching depth limits

Oiginally I tried to implement the code based on this Langchain example: https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/lats/lats.ipynb
. However that solution didn't work quite well for Ollama models. That's the origin of the following implementation.

## Core Components

### Tree Structure and Node Management

The implementation is built around a `Node` class that represents states in the search tree:

```python
class Node:
    def __init__(self, messages, response, reflection: Reflection, parent=None):
        self.messages = messages          # Conversation history
        self.response = response          # LLM response at this node
        self.reflection = reflection      # Quality evaluation
        self.parent = parent             # Tree structure
        self.children = []               # Child nodes
        self.value = 0                   # UCB statistics
        self.visits = 0
```

Each node keeps track of the conversation context, response quality metrics, and tree relationships needed for the search algorithm.

### Structured Evaluation System

LATS uses a structured reflection mechanism with Pydantic models:

```python
class Reflection(BaseModel):
    reflection: str = Field(description="Critique of response quality...")
    score: int = Field(description="Score from 0-100...")
    found_solution: bool = Field(description="Whether response solved the task...")
```

This gives us both qualitative feedback and quantitative scores to guide the tree search decisions.

### Upper Confidence Bound Selection

The algorithm uses UCB to balance exploration and exploitation:

```python
def upper_confidence_bound(self, exploration_weight=1.0):
    average_reward = self.value / self.visits
    exploration_term = math.sqrt(math.log(self.parent.visits) / self.visits)
    return average_reward + exploration_weight * exploration_term
```

This approach balances exploring promising new paths with leveraging known high-quality responses.

## LATS Workflow

### 1. Initial Response Generation

The process begins by generating a root response with tool access:

```python
def generate_initial_response(state: TreeState):
    # Generate response with potential tool calls
    ai_msg = initial_answer_chain.invoke({"input": query})
    
    # Execute any tool calls (e.g., web search)
    for tool_call in ai_msg.tool_calls:
        tool_msg = tool_node.invoke(tool_call)
        messages.extend(tool_msg['messages'])
    
    # Create reflection and establish root node
    reflection = reflection_chain.invoke({"question": query, "candidate": response})
    root = Node(messages, response=response, reflection=reflection)
```

### 2. Tree Expansion

The algorithm expands promising nodes by generating alternative options:

```python
def expand(state: TreeState):
    # SELECT: Find best leaf using UCB
    best_candidate = select(root)
    
    # EXPAND: Generate multiple alternatives
    new_candidates = generate_candidates(best_candidate.get_trajectory())
    
    # EVALUATE: Create reflections for each candidate
    for candidate in new_candidates:
        reflection = reflection_chain.invoke({"question": query, "candidate": candidate})
        child_node = Node(candidate, response, parent=best_candidate, reflection=reflection)
        best_candidate.children.append(child_node)
```

### 3. Termination Conditions

The search terminates when:
- A high-quality solution is found (`reflection.found_solution = True`)
- Maximum tree depth is reached
- Solution confidence exceeds threshold

## Tool Integration and External Knowledge

The implementation seamlessly integrates external tools like web search:

```python
# Search tool for accessing current information
search_tool = BraveSearch.from_api_key(api_key=api_key, search_kwargs={"count": 1})
tool_node = ToolNode(tools=[search_tool])

# LLM with tool binding
llm = llm.bind_tools(tools=tools)
```

This allows the agent to gather fresh information during the search process, which is especially useful for research queries.

## Configuration and Tuning

The tuning is done using a few configuration options:

```python
class Config:
    EXPLORATION_WEIGHT = 1.0    # Controls exploration vs exploitation
    MAX_TREE_HEIGHT = 5         # Depth limits
    NUM_CANDIDATES = 2          # Alternatives per expansion
    BUDGET = 1000              # Resource limits
```

You can adjust these parameters based on your specific use case and computational budget.

## LangGraph Integration

The implementation leverages LangGraph for workflow orchestration:

```python
builder = StateGraph(TreeState)
builder.add_node("start", generate_initial_response)
builder.add_node("expand", expand)
builder.add_conditional_edges("start", should_loop, ["expand", END])
builder.add_conditional_edges("expand", should_loop, ["expand", END])
```

This creates a solid state machine that handles the entire LATS process from start to finish.

## Use Cases and Benefits

LATS excels in scenarios requiring:

- **Research Reports**: Complex topics needing comprehensive coverage
- **Technical Documentation**: Multi-faceted explanations requiring accuracy
- **Creative Writing**: Exploring different narrative approaches
- **Problem Solving**: Mathematical or logical problems with multiple solution paths

## Performance Considerations

The algorithm balances quality improvements with computational costs:

- **Selective Expansion**: Only promising branches are explored further
- **Early Termination**: High-quality solutions end search immediately  
- **Resource Budgeting**: Configurable limits prevent excessive computation
- **Tool Optimization**: Strategic use of external resources

## Future Enhancements

Potential improvements include:

- **Parallel Evaluation**: Concurrent candidate assessment
- **Dynamic Exploration**: Adaptive exploration weights
- **Memory Integration**: Long-term context preservation
- **Multi-Modal Support**: Visual and audio content integration

## Implementation and Code

The complete Agent LATS implementation discussed in this blog post is available in my [genai_patterns repository](https://github.com/bvcmartins/genai_patterns/) on GitHub. The implementation demonstrates practical applications of LATS using LangGraph and provides examples for various use cases including research tasks and complex reasoning problems.

## Conclusion

Agent LATS is a significant step forward in AI reasoning, showing how tree search algorithms can systematically improve language model outputs. By combining structured evaluation, strategic exploration, and external tool integration, LATS helps AI agents produce more comprehensive and well-reasoned responses.

This implementation demonstrates how these concepts work in practice, providing a solid foundation for building more sophisticated AI reasoning systems. As language models continue to evolve, techniques like LATS will become increasingly important for tackling complex reasoning tasks.

---

*This blog post explores the Agent LATS implementation from the Jupyter notebook, highlighting key concepts and practical applications of tree search in AI reasoning.*