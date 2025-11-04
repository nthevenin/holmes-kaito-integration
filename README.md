# Holmes KAITO Integration

This package provides a complete integration of HolmesGPT with KAITO (Kubernetes AI Toolchain Operator) for AI-powered Kubernetes diagnostics using Azure's managed inference endpoints.

## Quick Start

### Prerequisites
- KAITO cluster with deployed models (qwen2.5-coder-7b-instruct recommended)
- kubectl access to your cluster
- Port forwarding to KAITO model endpoints

### Installation

1. **Clone Holmes and apply the integration**:
   ```bash
   git clone https://github.com/robusta-dev/holmesgpt.git
   cd holmesgpt
   git apply path/to/kaito-integration.patch
   ```

2. **Set up port forwarding** (in separate terminal):
   ```bash
   kubectl port-forward service/workspace-qwen2-5-coder-7b-instruct 8080:80
   ```

3. **Run Holmes with KAITO**:
   ```bash
   export OPENAI_API_BASE="http://localhost:8080/v1"
   export OPENAI_API_KEY="dummy" 
   export HOLMES_TOOL_CHOICE="required"
   
   holmes ask "what pods are running?" \
     --model="openai/qwen2.5-coder-7b-instruct" \
     --config super-minimal-config.yaml \
     --max-steps=7 \
     --refresh-toolsets
   ```

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

## Environment Variables

- **OPENAI_API_BASE**: Your KAITO model endpoint (e.g., http://localhost:8080/v1)
- **OPENAI_API_KEY**: Dummy value (KAITO doesn't validate)
- **HOLMES_TOOL_CHOICE**: Set to "required" for consistent tool calling

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