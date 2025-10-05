# React Agent with LangGraph

A simple ReAct (Reasoning and Acting) agent implementation using LangGraph that can perform mathematical operations and respond to user queries with streaming output.

## Overview

This project demonstrates how to build an AI agent that can:
- Perform basic mathematical operations (addition, subtraction, multiplication)
- Use tools dynamically based on user requests
- Maintain conversation context through state management
- Stream responses in real-time
- Make intelligent decisions about when to use tools vs. direct responses

## Features

- **Tool Integration**: Custom mathematical tools with automatic binding
- **ReAct Pattern**: The agent reasons about when to use tools and when to respond directly
- **Streaming Output**: Real-time response streaming with pretty printing
- **State Management**: Maintains conversation history using LangGraph's state system
- **Environment Configuration**: Secure configuration using environment variables
- **Conditional Logic**: Smart routing between tool usage and conversation end

## Prerequisites

- Python 3.8+
- OpenAI API access
- Valid OpenAI API key

## Installation

1. Clone the repository or copy the code files
2. Install required dependencies:
   ```bash
   pip install langchain-openai langgraph python-dotenv
   ```

3. Create a `.env` file in the project root:
   ```env
   OPENAI_API_KEY=your_openai_api_key_here
   OPENAI_BASE_URL=your_base_url_here  # Optional: if using custom endpoint
   ```

## Project Structure

```
app1/
├── react_agent.py          # Main ReAct agent implementation
├── .env                    # Environment variables (create this)
├── .env.example           # Example environment file
```

## Usage

### Running the ReAct Agent

```bash
python react_agent.py
```

The agent will process the example query: "Add 34 + 31, then multiply the result by 5, and tell me a joke"

### Example Output
```
================================ Ai Message =================================

I'll help you with that calculation and then tell you a joke!

First, let me add 34 + 31:
================================ Tool Message ===============================
Name: add
Content: 65

Now I'll multiply 65 by 5:
================================ Tool Message ===============================
Name: multiply
Content: 325

So, 34 + 31 = 65, and 65 × 5 = 325.

Now for a joke: Why don't scientists trust atoms? Because they make up everything! 😄
```

### Running the Memory Agent

```bash
python 2_memory_agent.py
```

This version allows interactive conversation and saves the history to `logging.txt`.

## Code Architecture

### Key Components

#### AgentState
```python
class AgentState(TypedDict):
    messages: Annotated[Sequence[BaseMessage], add_messages]
```
Defines the state structure with message history and automatic message aggregation.

#### Tools
Three mathematical functions decorated with `@tool`:
- `add(a, b)`: Addition function
- `subtract(a, b)`: Subtraction function  
- `multiply(a, b)`: Multiplication function

#### Core Functions

**model_call(state)**: Main agent node that:
- Adds system prompt for context
- Invokes the LLM with current state
- Returns updated state with response

**should_continue(state)**: Decision function that:
- Checks if the last message contains tool calls
- Routes to either "tools" or "end" based on the decision

**print_stream(stream)**: Utility function for:
- Pretty printing streaming responses
- Handling different message types

### Graph Flow

```
User Input → Agent (model_call) → Decision Point (should_continue)
                                       ↓
                                  Tool Calls? → Tools → Back to Agent
                                       ↓
                                     End
```

## Configuration

### Environment Variables

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `OPENAI_API_KEY` | Your OpenAI API key | Yes | None |
| `OPENAI_BASE_URL` | Custom API endpoint | No | OpenAI's default |

### Model Configuration

Currently configured for OpenAI's GPT models. Update the model name:

```python
model = ChatOpenAI(model="gpt-3.5-turbo").bind_tools(tools)  # or "gpt-4"
```

## Customization

### Adding New Tools

```python
@tool
def divide(a: int, b: int):
    """Division function that handles zero division"""
    if b != 0:
        return a / b
    else:
        return "Cannot divide by zero"

# Add to tools list
tools = [add, subtract, multiply, divide]
```

### Modifying the System Prompt

```python
def model_call(state: AgentState) -> AgentState:
    system_prompt = SystemMessage(content="Your custom system prompt here")
    # ... rest of the function
```

### Changing Input Processing

```python
# For different queries
inputs = {"messages": [("user", "Your custom query here")]}

# For interactive mode (see 2_memory_agent.py)
user_input = input("Enter your query: ")
while user_input != "exit":
    # Process input...
```

## Features Comparison

| Feature | react_agent.py | 2_memory_agent.py |
|---------|---------------|-------------------|
| Tool Usage | ✅ Mathematical tools | ❌ No tools |
| Interactive Mode | ❌ Single query | ✅ Continuous chat |
| Memory Persistence | ❌ Session only | ✅ File logging |
| Streaming Output | ✅ Pretty print | ✅ Console output |
| ReAct Pattern | ✅ Full implementation | ❌ Simple chat |

## Dependencies

```bash
langchain-openai>=0.1.0    # OpenAI integration
langgraph>=0.1.0           # Graph-based agent framework
python-dotenv>=1.0.0       # Environment variable management
```

## Troubleshooting

### Common Issues

1. **Incomplete Model Name**: 
   ```python
   # Fix this line in react_agent.py
   model = ChatOpenAI(model="gpt-3.5-turbo").bind_tools(tools)
   ```

2. **API Key Issues**: 
   - Verify `.env` file exists and contains valid API key
   - Check that `load_dotenv()` is called before using the key

3. **Import Errors**: 
   ```bash
   pip install --upgrade langchain-openai langgraph python-dotenv
   ```

4. **Tool Call Issues**:
   - Ensure tools are properly decorated with `@tool`
   - Verify tools are included in the `tools` list
   - Check that `bind_tools(tools)` is called on the model

### Debug Mode

Add debug output to trace execution:

```python
def model_call(state: AgentState) -> AgentState:
    print(f"Current state: {len(state['messages'])} messages")
    # ... rest of function
    print(f"Response: {response.content[:100]}...")
    return {"messages": [response]}
```

## Best Practices

1. **Environment Security**: Always use `.env` files for sensitive data
2. **Error Handling**: Add try-catch blocks for API calls
3. **Tool Documentation**: Write clear docstrings for tools
4. **State Management**: Keep state minimal and focused
5. **Testing**: Test tools independently before integration

## License

This project is for educational purposes. Please ensure you comply with OpenAI's usage policies when using their API.

## Contributing

Feel free to submit issues and enhancement requests!

## Changelog

- **v1.0**: Initial ReAct agent implementation
- **v1.1**: Added memory agent with conversation logging
- **v1.2**: Enhanced documentation and
