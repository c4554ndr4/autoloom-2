# Autoloom

**An intelligent AI writing companion that learns from your choices**

Autoloom is a sophisticated AI text generation system that helps you craft high-quality content through an iterative, guided process. Unlike traditional AI writing tools that generate once and stop, Autoloom creates multiple completions, scores them for quality, and builds upon your selections to maintain context and coherence throughout extended writing sessions.

## What Makes Autoloom Unique

**🧠 Iterative Generation**: Generate multiple completions simultaneously, select the best one, and continue building from there
**📈 Quality Scoring**: Automatic classification and ranking of generated text using configurable AI models  
**🔄 Full Context Awareness**: Each generation sees the complete conversation history for coherent, contextual continuations
**🎛️ Fine-Tuned Control**: Adjust temperature, token limits, generation counts, and timing to match your writing style
**⚡ Multi-Model Support**: Work with OpenAI (GPT-4, GPT-3.5 Turbo Instruct) and Hyperbolic (Llama-405B) models
**🖥️ Terminal-First Design**: Clean, keyboard-driven interface optimized for focused writing

## Two Powerful Tools

**LUI (Loom User Interface)** - The main interactive writing environment where you craft content through guided AI collaboration

**TUNI (TUning Interface)** - A specialized tool for creating fine-tuning datasets that help distinguish human from AI-generated text

## How It Works

1. **Start with a prompt** - Enter your initial text or idea
2. **Generate multiple options** - AI creates several different continuations 
3. **Review and score** - Each option is automatically rated for quality and coherence
4. **Select the best** - Choose your preferred continuation from the ranked results
5. **Build iteratively** - The selected text becomes part of the context for the next generation
6. **Maintain coherence** - Each new generation sees the full conversation history, ensuring consistency

## Core Capabilities

**🎯 Smart Generation**
- Multiple AI models (GPT-4, GPT-3.5 Turbo Instruct, Llama-405B)
- Configurable parameters (temperature, tokens, generation count)
- Full sequence context for coherent long-form content

**📊 Quality Assessment** 
- Automatic scoring and ranking of all generated options
- Configurable classification models for different content types
- Quality-first approach ensures only the best content continues

**⚙️ Fine-Tuning Tools**
- TUNI interface for creating human vs AI classification datasets
- Breakpoint analysis for comprehensive training data
- JSONL export for standard ML workflows

## Installation

### Prerequisites
- Python 3.8+
- Poetry (for dependency management)
- OpenAI API key
- Hyperbolic API key (for Llama models)

### Setup
```bash
# Clone the repository
git clone <repository-url>
cd autoloom

# Install dependencies
poetry install

# Activate virtual environment
poetry shell

# Create .env file with your API keys
cat > .env << EOF
OPENAI_API_KEY=your_openai_api_key_here
HYPERBOLIC_API_KEY=your_hyperbolic_api_key_here
DEFAULT_CLASSIFIER=gpt-4
EOF
```

## Usage

### Running LUI (Interactive Text Generation)
```bash
poetry run lui
```

**Interface Controls:**
- `Ctrl+C`: Quit with confirmation
- `Ctrl+S`: Show full completion history  
- `Ctrl+=`: Zoom in
- `Ctrl+-`: Zoom out

**Workflow:**
1. Enter your initial prompt
2. Select generation model (Llama-405b, GPT-4 Base, etc.)
3. Select classification model (Default GPT-4, GPT-4.1)
4. Adjust parameters (temperature, max tokens, generation count)
5. Generate and review scored completions
6. Select best completion to continue the sequence
7. Repeat for iterative generation

### Running TUNI (Fine-tuning Dataset Creation)
```bash
poetry run tuni
```

Enter text to generate human vs AI classification examples. Output is saved to `tunes/generated_examples.jsonl`.

### Environment Variables

Required:
- `OPENAI_API_KEY` - Your OpenAI API key
- `HYPERBOLIC_API_KEY` - Your Hyperbolic API key

Optional:
- `DEFAULT_CLASSIFIER` - Default classification model (defaults to "gpt-4")
- `CLASSIFIER_*` or `*_CLASSIFIER` - Custom classifier model definitions

## Architecture

### Generation Flow
1. **Input Processing**: User prompt or accumulated sequence
2. **Model Selection**: Route to appropriate API (OpenAI Completions or Hyperbolic)
3. **Batch Generation**: Generate multiple completions simultaneously  
4. **Classification**: Score all completions using selected classifier
5. **Ranking**: Sort by quality scores for user selection
6. **Iteration**: Selected completion becomes part of the growing sequence

### API Integration
- **Generation Models**: Use `/v1/completions` endpoint with `text` field responses
- **Classification Models**: Use `/v1/chat/completions` endpoint with message format
- **Special Handling**: GPT-4.1 uses "developer" role instead of "system" role
- **Error Handling**: Exponential backoff with model-specific retry counts

## Parameter Tuning Guide

### Understanding the Parameters

The interface now shows labeled parameter fields with helpful descriptions:

**Temperature (0.0-1.0)**
- **Low (0.0-0.3)**: Very focused, predictable output. Good for factual content.
- **Medium (0.4-0.7)**: Balanced creativity and coherence. Default for most use cases.
- **High (0.8-1.0)**: Highly creative, unpredictable. Good for creative writing.

**Max Tokens**
- **Short (50-100)**: Brief continuations, good for iterative building
- **Medium (100-300)**: Paragraph-length responses
- **Long (300-500)**: Extended content, but may cause context issues

**Generations (3-10)**
- More generations = better selection but higher cost/time
- Start with 5, increase if you want more variety

**Wait Time (0-30 seconds)**
- Time to review completions before auto-continuing
- 0 = manual selection only, 10+ = time to read and choose

### Troubleshooting Common Issues

**Repetitive/Circular Completions:**
- Increase temperature (try 0.8-0.9)
- Increase max tokens (try 200-300)
- Use shorter initial prompts to avoid context overload
- Try different generation models

**Incoherent/Off-topic Responses:**
- Decrease temperature (try 0.3-0.5)
- Check that your prompt is clear and specific
- Ensure full sequence isn't getting too long

**Poor Quality Scores:**
- Try different classifier models
- Review if the classifier matches your content type
- Check that completions aren't cut off mid-sentence

## Model Support

### Generation Models
| Model | Provider | API | Context Window |
|-------|----------|-----|---------------|
| meta-llama/Meta-Llama-3.1-405B | Hyperbolic | Completions | Large |
| gpt-4-base | OpenAI | Completions | 8K |
| gpt-3.5-turbo-instruct | OpenAI | Completions | 4K |

### Classification Models  
| Model | Provider | API | Special Notes |
|-------|----------|-----|---------------|
| gpt-4 | OpenAI | Chat | Default classifier |
| gpt-4.1 | OpenAI | Chat | Uses "developer" role |

---

## FAQ

### Q: What is the basic classifier doing? Is it just a system prompt or a fine-tune?

**A:** The classifier uses a simple system prompt, not a fine-tuned model. It sends this instruction: *"You are a classifier. Your task is to rate the quality and coherence of text on a scale from 0-100. Respond with ONLY a number, no explanation."* Then it passes the generated text and expects back a numerical score (0-100). This is a zero-shot approach using the base model's understanding of text quality.

### Q: Do the completions attend to all the text that came before?

**A:** Yes, the completions API attends to all the text that came before. The system passes the entire prompt string to the API, and the `max_tokens` parameter only controls response length, not input truncation. The model processes the full prompt context up to its maximum context window.

### Q: What are the numbers we can fork?

**A:** Key configurable parameters include:

**UI Configurable (now labeled in interface):**
- **Temperature**: `0.7` (range 0.0-1.0)
  - Controls randomness/creativity of responses
  - 0.0 = focused, deterministic output
  - 1.0 = highly creative, unpredictable output
  - Sweet spot: 0.6-0.8 for most use cases
  
- **Max Tokens**: `100` (response length limit)
  - Maximum number of tokens the model can generate
  - Higher = longer completions (but may hit context limits)
  - Typical ranges: 50-200 for short continuations, 200-500 for longer content
  
- **Generations**: `5` (completions per round)
  - How many different completions to generate simultaneously
  - More = better selection but slower/more expensive
  - Recommended: 3-10 depending on use case
  
- **Wait Time**: `10` seconds (auto-continue delay)
  - Time before automatically continuing with selected completion
  - Gives you time to review and choose manually
  - Set to 0 to disable auto-continue

**Advanced Parameters (hardcoded):**
- **top_p**: `0.9` (nucleus sampling)
  - Controls diversity by only sampling from top X% probability tokens
  - Lower = more focused, higher = more diverse
- **Classifier temperature**: `0` (deterministic scoring)
- **Retry logic**: 8 attempts for GPT models, 5 for others
- **Timeouts**: 120s for GPT completions, 60s for others

**TUNI Parameters:**
- **max_tokens**: `5` (fine-tuning example length)
- **breakpoints_per_doc**: `3` (samples per document)
- **ai_completions_per_breakpoint**: `5` (AI examples per sample)

### Q: Does the loom attend to all of the text? Or just the last completion?

**A:** As of the latest version, the loom attends to the **full sequence** of completions, not just the last one. 

**Previous Behavior (Before Update):**
- Gen 1: `"Write a story about"` → `" a brave knight"`
- Gen 2: `" a brave knight"` → `" who discovered"`  
- Gen 3: `" who discovered"` → `" a magical sword"`

**Current Behavior (After Update):**
- Gen 1: `"Write a story about"` → `" a brave knight"`
- Gen 2: `"Write a story about a brave knight"` → `" who discovered"`
- Gen 3: `"Write a story about a brave knight who discovered"` → `" a magical sword"`

Each generation now has the full context of the entire conversation, enabling much more coherent and contextually aware continuations.

### Q: What improvements does Autoloom-2 have over the original?

**A:** Autoloom-2 includes several key enhancements:
- **Full sequence context**: Each generation now sees the entire conversation history, not just the last completion
- **Improved parameter interface**: Clear labels and compact layout for all generation parameters  
- **Better error handling**: Enhanced retry logic and timeout management
- **Cleaner UI**: Streamlined terminal interface with proper parameter visibility
- **Enhanced documentation**: Comprehensive README with troubleshooting guides

---

## Development

### Project Structure
```
autoloom/
├── lui/                    # Loom User Interface
│   ├── main.py            # LUI entry point
│   ├── ui/                # Terminal UI components  
│   │   ├── app.py         # Main application
│   │   ├── generation_manager.py  # Generation workflow
│   │   └── components/    # UI widgets
│   └── models/            # AI model interfaces
│       ├── generator.py   # Text generation
│       └── classifier.py  # Quality classification
├── tuni/                  # TUning Interface
│   ├── main.py           # TUNI entry point
│   ├── tuner.py          # Fine-tuning dataset logic
│   └── hyper_api.py      # Hyperbolic API client
├── tests/                # Test files
├── tunes/                # Generated datasets
└── pyproject.toml        # Poetry configuration
```

### Testing
```bash
# Test sequence logic
python test_sequence_logic.py
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable  
5. Submit a pull request

## License

[Add license information]

## Support

For issues, questions, or contributions, please [open an issue](repository-issues-url) on the project repository.