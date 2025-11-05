# Holmes KAITO Integration

This package provides a complete integration of HolmesGPT with KAITO (Kubernetes AI Toolchain Operator) for AI-powered Kubernetes diagnostics using Azure's managed inference endpoints.

## Quick Start

### Prerequisites
- KAITO cluster with deployed models (qwen2.5-coder-7b-instruct recommended)
- kubectl access to your cluster
- Python 3.8+ with pip and virtualenv support
- Port forwarding to KAITO model endpoints

### Installation

1. **Clone Holmes and apply the integration**:
   ```bash
   git clone https://github.com/robusta-dev/holmesgpt.git
   cd holmesgpt
   git apply path/to/kaito-integration.patch
   ```

2. **Set up Python virtual environment** (REQUIRED for code changes):
   ```bash
   # Create virtual environment
   python3 -m venv .venv
   
   # Activate virtual environment
   source .venv/bin/activate  # On Linux/macOS
   # .venv\Scripts\activate   # On Windows
   ```

3. **Install Holmes in editable mode** (CRITICAL for patch to work):
   ```bash
   pip install -e .
   ```

4. **Set up port forwarding** (in separate terminal):
   ```bash
   kubectl port-forward service/workspace-qwen2-5-coder-7b-instruct 8080:80
   ```

5. **Run Holmes with KAITO**:
   ```bash
   # Set environment variables (REQUIRED)
   export OPENAI_API_BASE="http://localhost:8080/v1"
   export OPENAI_API_KEY="dummy" 
   export HOLMES_TOOL_CHOICE="required"
   
   # Run Holmes (ensure you're in the activated virtual environment)
   holmes ask "what pods are running?" \
     --model="openai/qwen2.5-coder-7b-instruct" \
     --config super-minimal-config.yaml \
     --max-steps=7 \
     --refresh-toolsets
   ```

## Important Setup Notes

### Python Environment Requirements
- **Virtual Environment**: MANDATORY - Isolates dependencies and ensures patched code is used
- **Editable Installation**: `pip install -e .` is REQUIRED for the patch modifications to take effect
- **Environment Activation**: Must activate virtual environment (`source .venv/bin/activate`) before each Holmes session

### Environment Variables
All three environment variables are REQUIRED for proper operation:
- **OPENAI_API_BASE**: Points to your KAITO model endpoint
- **OPENAI_API_KEY**: Set to "dummy" (KAITO doesn't validate but Holmes requires it)
- **HOLMES_TOOL_CHOICE**: Must be "required" for consistent tool calling behavior

## What's Included

- **kaito-integration.patch**: Complete code modifications for KAITO compatibility
- **super-minimal-config.yaml**: Optimized Holmes configuration for KAITO
- **Documentation**: Detailed setup and usage instructions

## Key Features

- ✅ **Tool calling optimization** for KAITO models
- ✅ **Anti-JSON enforcement** for clean natural language responses
- ✅ **Conciseness controls** for faster inference
- ✅ **Minimal toolset configuration** for focused diagnostics
- ✅ **Environment variable configuration** for easy model switching

## Code Changes

The patch modifies 5 core Holmes files:
- `holmes/core/llm.py` - Tool choice logic improvements
- `holmes/core/tool_calling_llm.py` - Environment variable support
- `holmes/core/prompt.py` - Anti-JSON and conciseness enforcement
- `holmes/plugins/prompts/generic_ask.jinja2` - Response guidelines
- `holmes/plugins/toolsets/investigator/core_investigation.py` - Disable TodoWrite

Plus adds:
- `super-minimal-config.yaml` - Minimal Holmes configuration

## Tested Configuration

This integration has been tested with:
- **Model**: qwen2.5-coder-7b-instruct
- **Toolsets**: kubernetes/core, aks/core
- **Max Steps**: 7 (optimal balance)
- **KAITO Version**: Latest stable

## Support

For issues or questions about this integration, please refer to the detailed documentation in `KAITO_INTEGRATION.md`.

---

*This integration enables HolmesGPT to work seamlessly with KAITO's OpenAI-compatible endpoints for Kubernetes diagnostics.*