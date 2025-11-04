# HolmesGPT KAITO Integration Guide

This guide details how to integrate KAITO with HolmesGPT for Kubernetes diagnostics, enabling AI-powered cluster analysis through Azure's managed inference endpoints.

## Overview

This integration modifies HolmesGPT's tool calling behavior and prompts for optimal performance with KAITO's model characteristics. The setup involves applying code changes and using a minimal configuration optimized for KAITO models.

## Prerequisites

1. **KAITO Installation** - Follow the standard KAITO installation guide for your Azure environment
2. **GPU Cluster Configuration** - Set up an AKS cluster with GPU-enabled nodes
3. **Model Compatibility** - Deploy a KAITO model with robust function calling capabilities (qwen2.5-coder-7b-instruct recommended)
4. **Network Access** - Ensure kubectl access to your AKS cluster for port forwarding

## Installation Steps

### 1. Apply the Holmes Integration

```bash
# Clone fresh Holmes repository
git clone https://github.com/robusta-dev/holmesgpt.git
cd holmesgpt

# Apply the KAITO integration patch
git apply path/to/kaito-integration.patch

# Install Holmes dependencies
poetry install
```

### 2. Port Forwarding Setup

Set up port forwarding for your KAITO model (keep this running):

```bash
kubectl port-forward service/workspace-qwen2-5-coder-7b-instruct 8080:80
```

### 3. Test Model Availability

Verify your KAITO model is accessible:

```bash
# Test model endpoint
curl -s "http://localhost:8080/v1/models" | jq .

# Test basic functionality
curl -X POST "http://localhost:8080/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer dummy-key" \
  -d '{
    "model": "qwen2.5-coder-7b-instruct",
    "messages": [{"role": "user", "content": "Say hello"}],
    "max_tokens": 50
  }'
```

## Usage

### Basic Command

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

### Environment Variables Explained

- **OPENAI_API_BASE**: Points to your KAITO model endpoint
- **OPENAI_API_KEY**: Dummy value (KAITO doesn't validate keys)
- **HOLMES_TOOL_CHOICE**: Forces consistent tool calling behavior

### Command Parameters

- **--model**: Use "openai/" prefix for litellm provider detection
- **--config**: Path to the included minimal configuration
- **--max-steps**: Limits tool calling iterations (7 is optimal)
- **--refresh-toolsets**: Reloads configurations

## Code Modifications

### Core Functional Changes

#### 1. holmes/core/llm.py - Tool Choice Logic Fix
```python
# Before: if tools and len(tools) > 0 and tool_choice == "auto":
# After:  if tools and len(tools) > 0 and tool_choice:
```
Enables `HOLMES_TOOL_CHOICE="required"` to work properly.

#### 2. holmes/core/tool_calling_llm.py - Environment Variable Support
```python
# Before: tool_choice = "auto" if tools else None
# After:  tool_choice = os.environ.get("HOLMES_TOOL_CHOICE", "auto") if tools else None
```
Allows runtime configuration of tool calling behavior.

#### 3. holmes/plugins/toolsets/investigator/core_investigation.py - Disable TodoWrite
```python
enabled=False,    # Modified for KAITO compatibility
is_default=False  # Modified for KAITO compatibility
```
Disables TodoWrite system that interferes with clean responses.

### Core Prompting Changes

#### 1. holmes/core/prompt.py - Anti-JSON and Conciseness Enforcement
Adds KAITO-specific prompting rules:
- Anti-JSON enforcement for natural language responses
- Conciseness requirements for faster inference
- Precise response guidelines

#### 2. holmes/plugins/prompts/generic_ask.jinja2 - Critical Response Guidelines
Adds CRITICAL sections enforcing:
- Natural language responses instead of JSON
- Concise diagnostic answers
- Step-by-step response workflow
- Precise numerical accuracy

## Configuration

### Minimal Toolset Configuration

The included `super-minimal-config.yaml` enables only essential toolsets:
- **kubernetes/core**: Basic Kubernetes operations
- **aks/core**: Azure-specific Kubernetes features

This minimal configuration reduces complexity and improves KAITO model performance.

## Optimization Notes

### Max Steps Configuration
- **Too low (1-3)**: May not gather enough information
- **Too high (15+)**: Can cause tool calling loops
- **Sweet spot (5-9)**: Thorough investigation, prevents loops

### Model Performance Tips
- Use `tool_choice="required"` for consistent behavior
- Monitor response times and adjust max-steps accordingly
- Test different KAITO models for your specific use cases

## Troubleshooting

### Common Issues

**Responses too verbose**: Strengthen conciseness prompts or reduce max-steps
**Tool calling fails**: Verify `HOLMES_TOOL_CHOICE="required"` and model compatibility
**Incomplete answers**: Increase max-steps or check toolset availability
**JSON responses**: Review anti-JSON prompt placement

### Debug Steps

1. Verify port forwarding is active
2. Test model endpoint directly with curl
3. Check environment variables are set correctly
4. Ensure patch was applied successfully
5. Validate toolset configuration

## Performance Characteristics

### Tested Environment
- **Model**: qwen2.5-coder-7b-instruct
- **Cluster**: AKS with GPU nodes
- **Average Response Time**: 15-30 seconds
- **Tool Calling Success Rate**: >95%

### Resource Requirements
- **Model Memory**: ~8GB GPU memory
- **Network**: Stable connection to cluster
- **CPU**: Standard development machine

## Advanced Configuration

### Custom Model Support
To use different KAITO models:
1. Update the model name in commands
2. Adjust port forwarding service name
3. Test tool calling compatibility
4. Fine-tune max-steps if needed

### Extended Toolsets
To enable additional Holmes toolsets:
1. Modify `super-minimal-config.yaml`
2. Test with your specific cluster setup
3. Monitor performance impact
4. Adjust max-steps accordingly

---

*This integration provides a production-ready setup for using HolmesGPT with KAITO's managed inference endpoints.*