# Drafter Agent - Interactive Document Writing Assistant

A LangGraph-powered document writing assistant that helps users create, update, and save text documents through natural language interactions.

## Overview

Drafter is an AI-powered writing assistant that can:
- **Create Documents**: Start with blank documents or existing content
- **Update Content**: Modify documents based on natural language instructions
- **Save Files**: Automatically save completed documents to text files
- **Interactive Workflow**: Continuous conversation until document is saved
- **State Management**: Maintains document content throughout the session

## Features

- **Natural Language Document Editing**: Tell the AI what you want to write or change
- **Real-time Content Updates**: See document changes immediately
- **Auto-save Functionality**: Documents are saved as `.txt` files
- **Interactive CLI Interface**: Step-by-step conversation flow
- **Error Handling**: Graceful handling of file operations
- **Session Persistence**: Document content maintained throughout interaction

## Prerequisites

- Python 3.8+
- OpenAI API key
- Required Python packages

## Installation

1. **Install dependencies**:
   ```bash
   pip install langchain-openai langgraph python-dotenv
   ```

2. **Set up environment variables**:
   ```bash
   # Create .env file
   echo "API_KEY=your_openai_api_key_here" > .env
   ```

## Usage

### Basic Usage
```bash
python drafter_agent.py
```

### Example Interaction Flow

```
===== DRAFTER =====

🤖 AI: I'm ready to help you update a document. What would you like to create?

👤 USER: Write a short story about a robot learning to paint

🤖 AI: I'll create a short story about a robot learning to paint for you.
🔧 USING TOOLS: ['update']

👤 USER: Make it more emotional and add a happy ending

🤖 AI: I'll make the story more emotional and add a happy ending.
🔧 USING TOOLS: ['update']

👤 USER: Save this as "robot_story"

🤖 AI: I'll save the document for you now.
🔧 USING TOOLS: ['save']

💾 Document has been saved to: robot_story.txt

===== DRAFTER FINISHED =====
```

## Architecture

### Core Components

#### 1. Global State Management
```python
document_content = ""  # Stores current document content globally
```

#### 2. Document Tools

**Update Tool**:
```python
@tool
def update(content: str) -> str:
    """Update the document with the provided content"""
```
- Replaces entire document content
- Returns confirmation with current content

**Save Tool**:
```python
@tool
def save(filename: str) -> str:
    """Save the current document to a text file"""
```
- Automatically adds `.txt` extension if missing
- Saves to current directory
- Triggers conversation end

#### 3. Agent Logic

**our_agent()**: Main conversation handler
- Provides dynamic system prompts with current document state
- Handles user input collection
- Manages conversation flow

**should_continue()**: Flow control
- Continues conversation by default
- Ends when document is saved (detects "saved" + "document" in tool messages)

### Workflow

```
Start → Agent (Get User Input) → Tools (Update/Save) → Continue?
                                                           ↓
                                                      Yes → Agent
                                                           ↓
                                                      No → End
```

## Configuration

### Environment Variables
```bash
# .env file
API_KEY=your_openai_api_key_here
MODEL=gpt-3.5-turbo          # Optional, defaults to gpt-3.5-turbo
BASE_URL=your_custom_url     # Optional, for custom endpoints
```

### Model Configuration
```python
model = ChatOpenAI(
    model=os.getenv("MODEL", "gpt-3.5-turbo"),
    api_key=os.getenv("API_KEY"),
    base_url=os.getenv("BASE_URL")
).bind_tools(tools)
```

## Customization

### Adding New Tools

```python
@tool
def append(content: str) -> str:
    """Append content to the existing document"""
    global document_content
    document_content += f"\n{content}"
    return f"Content appended! Current document:\n{document_content}"

# Add to tools list
tools = [update, save, append]
```

### Modifying System Prompt

```python
def our_agent(state: AgentState) -> AgentState:
    system_prompt = SystemMessage(content=f"""
    You are a specialized technical writer. 
    Focus on creating clear, structured documents.
    
    Current document: {document_content}
    """)
    # ... rest of function
```

### Custom File Formats

```python
@tool
def save_markdown(filename: str) -> str:
    """Save document as markdown file"""
    global document_content
    
    if not filename.endswith(".md"):
        filename = f"{filename}.md"
    
    # Add markdown formatting logic here
    formatted_content = f"# Document\n\n{document_content}"
    
    with open(filename, 'w') as file:
        file.write(formatted_content)
    
    return f"Markdown document saved as {filename}"
```

## Key Features Explained

### Interactive Input Collection
- Uses `input()` for real-time user interaction
- Displays formatted conversation with emojis
- Shows tool usage for transparency

### Document State Management
- Global variable maintains document across tool calls
- System prompt updates with current content
- Tools can access and modify shared state

### Automatic Conversation End
- Detects when document is saved
- Automatically terminates conversation
- Prevents infinite loops

### Error Handling
```python
try: 
    with open(filename, 'w') as file:
        file.write(document_content)
except Exception as e:
    return f"Error saving document: {str(e)}"
```

## Use Cases

1. **Creative Writing**: Stories, poems, articles
2. **Technical Documentation**: Guides, manuals, specifications
3. **Business Documents**: Reports, memos, proposals
4. **Educational Content**: Lessons, summaries, notes
5. **Code Documentation**: README files, API docs

## Troubleshooting

### Common Issues

1. **Environment Variables**:
   ```bash
   # Check if .env file exists
   ls -la .env
   
   # Verify content
   cat .env
   ```

2. **Tool Not Executing**:
   - Check tool is in `tools` list
   - Verify `bind_tools(tools)` is called
   - Ensure tool has proper docstring

3. **Conversation Not Ending**:
   - Check `should_continue()` logic
   - Verify save tool returns correct message format
   - Ensure "saved" and "document" are in tool response

4. **File Save Issues**:
   - Check write permissions in current directory
   - Verify filename is valid
   - Check for disk space

### Debug Mode

Add debugging to trace execution:

```python
def our_agent(state: AgentState) -> AgentState:
    print(f"DEBUG: Current document length: {len(document_content)}")
    print(f"DEBUG: State messages: {len(state['messages'])}")
    # ... rest of function
```

## Best Practices

1. **Document Backup**: Consider implementing auto-backup functionality
2. **Version Control**: Track document changes for undo functionality
3. **Input Validation**: Validate user input before processing
4. **File Naming**: Implement filename sanitization
5. **Content Limits**: Add document size limits for performance

## Dependencies

```
langchain-openai>=0.1.0    # OpenAI integration
langgraph>=0.1.0           # Graph workflow framework
python-dotenv>=1.0.0       # Environment variable management
```

## Security Considerations

- **File Permissions**: Documents are saved with default permissions
- **Directory Traversal**: No path validation (consider adding)
- **Content Sanitization**: Raw content is saved without filtering
- **API Key Security**: Keep `.env` files out of version control

## Future Enhancements

- [ ] Multiple document support
- [ ] Document templates
- [ ] Export to different formats (PDF, DOCX)
- [ ] Version history tracking
- [ ] Collaborative editing features
- [ ] Document encryption

## License

Educational use only. Follow OpenAI's usage policies when using their API.

## Contributing

Feel
