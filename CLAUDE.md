# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

This is Autoloom, an AI text generation and classification system with two main applications:

- **LUI** (`lui/`) - Loom User Interface: A terminal UI for interactive text generation with multiple AI models and quality scoring
- **TUNI** (`tuni/`) - TUning Interface: A tool for generating fine-tuning datasets to distinguish human vs AI-generated text

## Development Commands

### Environment Setup
```bash
# Install dependencies
poetry install

# Activate virtual environment
poetry shell
```

### Running Applications
```bash
# Run LUI (terminal UI for text generation)
poetry run lui

# Run TUNI (fine-tuning dataset generator) 
poetry run tuni
```

### Environment Variables Required
Create a `.env` file with:
- `OPENAI_API_KEY` - Required for classification and OpenAI models
- `HYPERBOLIC_API_KEY` - Required for Hyperbolic models (Llama)
- `DEFAULT_CLASSIFIER` - Optional, defaults to "gpt-4"
- Custom classifier models can be added with `CLASSIFIER_*` or `*_CLASSIFIER` prefixes

## Architecture Overview

### LUI Application Flow
1. **Main Entry** (`lui/main.py`) → **App** (`lui/ui/app.py`)
2. **GenerationManager** manages the generation workflow
3. **Generator** (`lui/models/generator.py`) handles multiple AI APIs:
   - OpenAI Completions API (GPT-4 Base, GPT-3.5 Turbo Instruct) 
   - Hyperbolic API (Llama models)
4. **Classifier** (`lui/models/classifier.py`) scores generation quality using OpenAI Chat API
5. **UI Components** provide terminal interface with Textual framework

### TUNI Fine-tuning System
1. **Main Entry** (`tuni/main.py`) → **Tuner** (`tuni/tuner.py`)
2. **ContrastContext** manages breakpoint-based dataset generation
3. **HyperBaseClient** (`tuni/hyper_api.py`) handles API communications
4. Generates JSONL files with human/AI text classification examples

### Key API Patterns
- **Generation Models**: Use completions API (`/v1/completions`) with `text` field responses
- **Classification Models**: Use chat completions API (`/v1/chat/completions`) with message format
- **GPT-4.1 Special Handling**: Uses "developer" role instead of "system" role for prompts
- **Error Handling**: Exponential backoff with model-specific retry counts and timeouts

### Data Flow
- Generated text → Quality scoring → Ranking → User selection
- Fine-tuning: Text breakpoints → AI completions → Human/AI classification examples → JSONL output

## Model Support

### Generation Models (Completions API)
- `meta-llama/Meta-Llama-3.1-405B` (Hyperbolic)
- `gpt-4-base` (OpenAI) 
- `gpt-3.5-turbo-instruct` (OpenAI)

### Classification Models (Chat API)
- `gpt-4` series (OpenAI)
- `gpt-4.1` (OpenAI, uses developer role)
- Custom models via environment variables

## File Structure Notes
- `tunes/` - Generated fine-tuning datasets
- `tests/` - Test directory (minimal setup)
- UI components use Textual CSS styling (`styles.css`)
- Async/await patterns throughout for API handling