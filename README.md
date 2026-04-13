# Writer-Reviewer Multi-Agent Workflow

A collaborative content creation workflow powered by Python Agent Framework SDK, featuring a Writer agent that creates content and a Reviewer agent that provides actionable feedback.

## 🎯 What This Does

```
User Request
    ↓
[Writer Agent] → Creates polished content
    ↓
[Reviewer Agent] → Provides concise feedback
    ↓
Output: Refined content + feedback (as plain text)
```

## Quick Start

### Prerequisites
- Python 3.9+
- GitHub PAT (Personal Access Token) for GitHub models, OR
- Azure subscription with Foundry + OpenAI deployment

### Installation

1. **Clone or open this project**
   ```bash
   cd writer_reviewer_workflow
   ```

2. **Create virtual environment**
   ```bash
   python -m venv venv
   
   # Windows
   venv\Scripts\activate
   
   # macOS/Linux
   source venv/bin/activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Configure environment**
   ```bash
   cp .env.example .env
   ```
   
   Edit `.env`:
   - **For GitHub models**: Add your GitHub PAT token
   - **For Foundry**: Add Foundry endpoint and model name

### Run Interactively

Test the workflow locally:

```bash
python workflow.py
```

Output:
```
============================================================
WRITER-REVIEWER WORKFLOW
============================================================

User Request: Write a brief blog post about remote work productivity

------------------------------------------------------------

WRITER:
[Generated content from writer agent...]

REVIEWER:
[Feedback from reviewer agent...]
============================================================
```

### Run as HTTP Server

Start the HTTP server:

```bash
python main.py
```

Server starts on `http://localhost:8088`

**Make a request:**

```bash
curl -X POST http://localhost:8088/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "user",
        "content": "Write marketing copy for an eco-friendly water bottle"
      }
    ]
  }'
```

## 🔧 Configuration

### Using GitHub Models (Default)

1. Get a GitHub PAT: https://github.com/settings/tokens
2. Add to `.env`:
   ```env
   GITHUB_TOKEN=github_pat_xxxxxxxxxxxxx
   MODEL=gpt-4o
   ```

### Using Microsoft Foundry

1. Set up Foundry project with Azure OpenAI model
2. Update `.env`:
   ```env
   FOUNDRY_PROJECT_ENDPOINT=https://your-project.api.azdeveloper.ai
   FOUNDRY_MODEL=gpt-4-turbo
   AZURE_SUBSCRIPTION_ID=your-sub-id
   ```
3. Update imports in `workflow.py` to use `FoundryChatClient` instead of `OpenAIChatClient`

## 🐛 Debugging

### In VSCode

1. **Run interactive workflow with debugging:**
   - Open `workflow.py`
   - Press `F5` → Select "Debug Interactive Mode"
   - Set breakpoints as needed

2. **Debug HTTP Server:**
   - Press `F5` → Select "Debug HTTP Server"
   - Inspector opens automatically
   - Send requests to `http://localhost:8088`

### From Terminal

```bash
# Run with verbose output
python -m debugpy --listen 127.0.0.1:5679 workflow.py
```

## 📊 Tracing & Observability

Enable OpenTelemetry tracing for monitoring:

```python
from tracing_config import setup_tracing

setup_tracing(
    service_name="writer-reviewer-workflow",
    otel_endpoint="http://localhost:4317",
)
```

Or set environment variables:
```bash
OTEL_SDK_DISABLED=false
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
```

## 🚀 Deploy to Microsoft Foundry

### Prerequisites
- Azure subscription
- Azure Developer CLI (`azd`)
- Foundry project with OpenAI deployment

### Steps

1. **Initialize deployment (first time only)**
   ```bash
   azd init
   ```

2. **Validate configuration**
   ```bash
   azd validate
   ```

3. **Deploy**
   ```bash
   azd up
   ```

4. **Monitor**
   - Logs available in Azure Portal
   - Monitor agent performance in Foundry dashboard

5. **Call deployed endpoint**
   ```bash
   curl -X POST https://your-foundry-endpoint/v1/chat/completions \
     -H "Authorization: Bearer $TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"messages": [{"role": "user", "content": "Your content request"}]}'
   ```

## 🏗️ Project Structure

```
.
├── workflow.py           # Writer & Reviewer executors, workflow builder
├── main.py              # HTTP server entry point
├── tracing_config.py    # OpenTelemetry setup
├── requirements.txt     # Python dependencies
├── .env.example         # Configuration template
├── .vscode/
│   ├── launch.json      # Debug configurations
│   └── tasks.json       # VSCode tasks
└── README.md            # This file
```

## 📝 How It Works

### Writer Agent
- **Role**: Content creator
- **Input**: User request + conversation history
- **Output**: Well-structured, engaging content
- **Instructions**: Focus on clarity, accuracy, and professionalism

### Reviewer Agent
- **Role**: Quality assurance & feedback provider
- **Input**: Full conversation with writer's content
- **Output**: Concise, actionable feedback
- **Focus**: Clarity, accuracy, structure, tone

### Workflow
1. Receives user prompt
2. Writer processes and generates content
3. Reviewer analyzes complete conversation
4. Both outputs streamed back to client
5. Final result: Refined content + feedback

## 🔌 API Reference

### HTTP Endpoint

**URL**: `http://localhost:8088/v1/chat/completions`

**Method**: `POST`

**Request**:
```json
{
  "messages": [
    {
      "role": "user",
      "content": "Your content request here"
    }
  ]
}
```

**Response** (streaming):
```json
{
  "choices": [
    {
      "delta": {
        "content": "Writer: [streaming content]"
      }
    },
    {
      "delta": {
        "content": "Reviewer: [streaming feedback]"
      }
    }
  ]
}
```

## 🤝 Contributing

To extend the workflow:

1. **Add new executor**:
   ```python
   class Editor(Executor):
       @handler
       async def handle(self, messages: list[Message], ctx: WorkflowContext) -> None:
           # Implementation
           await ctx.yield_output(...)
   ```

2. **Add to workflow**:
   ```python
   workflow = (
       WorkflowBuilder(start_executor=writer)
       .add_edge(writer, reviewer)
       .add_edge(reviewer, editor)  # New edge
       .build()
   )
   ```

## ⚙️ Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `GITHUB_TOKEN` | (required) | GitHub PAT for models access |
| `MODEL` | `gpt-4o` | Model ID from GitHub models |
| `MODEL_ENDPOINT` | `https://models.github.ai/inference/` | GitHub models endpoint |
| `FOUNDRY_PROJECT_ENDPOINT` | (optional) | Foundry project endpoint for Foundry models |
| `FOUNDRY_MODEL` | (optional) | Model deployment name in Foundry |
| `OTEL_SDK_DISABLED` | `false` | Enable/disable tracing |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | (optional) | OpenTelemetry collector endpoint |

## 📚 Resources

- [Microsoft Agent Framework Documentation](https://github.com/microsoft/agent-framework)
- [GitHub Models](https://github.com/marketplace/models)
- [Microsoft Foundry](https://aka.ms/foundry)
- [OpenTelemetry Python](https://opentelemetry.io/docs/instrumentation/python/)

## 🆘 Troubleshooting

| Issue | Solution |
|-------|----------|
| `ModuleNotFoundError: No module named 'agent_framework'` | Run `pip install -r requirements.txt` |
| `ImportError: No module named 'openai'` | Run `pip install openai` |
| `Connection error to GitHub models` | Verify `GITHUB_TOKEN` is valid and has model access |
| `Server won't start on port 8088` | Check if port is in use: `netstat -an \| grep 8088` |
| `Tracing not working` | Ensure OpenTelemetry is installed: `pip install opentelemetry-api opentelemetry-sdk opentelemetry-exporter-otlp` |

## 📄 License

MIT

## 🙋 Questions?

Refer to `.github/copilot-instructions.md` for development guidelines and debugging tips.
