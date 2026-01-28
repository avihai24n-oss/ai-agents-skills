# Local LLM Model Capability Matrix

Comprehensive reference for model selection in air-gapped/local environments.

## Quick Reference

### By Task Type

| Task | Tier 1 (Best) | Tier 2 (Good) | Tier 3 (Basic) |
|------|---------------|---------------|----------------|
| **Coding** | qwen3-coder-30b, deepseek-coder-v2 | codellama-34b, qwen2.5-coder-14b | codellama-7b, deepseek-coder-6.7b |
| **Reasoning** | qwen3-235b, deepseek-r1 | qwen2.5-72b, llama-3.3-70b | qwen2.5-32b, llama-3.1-8b |
| **Analysis** | qwen3-coder-30b + Serena | codellama-34b-instruct | - |
| **Documentation** | qwen2.5-72b, llama-3.3-70b | mistral-nemo-12b | mistral-7b |

### By VRAM Requirement

| VRAM | Models |
|------|--------|
| **5-8 GB** | codellama-7b, deepseek-coder-6.7b, mistral-7b, llama-3.1-8b, phi-3-medium |
| **10-20 GB** | qwen2.5-coder-14b, starcoder2-15b, qwen3-coder-30b, codellama-34b, qwen2.5-32b |
| **40-48 GB** | codellama-70b, llama-3.3-70b, qwen2.5-72b, deepseek-coder-v2, mixtral-8x22b |
| **140+ GB** | qwen3-235b |

---

## Coding Specialists

### Tier 1 - Best Performance

#### qwen3-coder-30b
- **Context Window**: 131,072 tokens
- **VRAM Required**: 20 GB
- **Strengths**: All-around coding excellence, large context
- **Best For**: Complex multi-file refactoring, full codebase understanding
- **Quantization Options**: Q4_K_M (12GB), Q5_K_M (15GB), Q8_0 (20GB)

#### deepseek-coder-v2
- **Context Window**: 128,000 tokens
- **VRAM Required**: 48 GB (MoE - 236B params, 21B active)
- **Strengths**: State-of-the-art code generation, reasoning
- **Best For**: Complex algorithms, architectural decisions
- **Note**: MoE architecture - efficient despite large param count

#### codellama-70b
- **Context Window**: 100,000 tokens
- **VRAM Required**: 40 GB
- **Strengths**: Strong on legacy languages (C/C++, Java)
- **Best For**: Enterprise codebases, system programming
- **Variants**: base, instruct, python

### Tier 2 - Good Balance

#### codellama-34b
- **Context Window**: 100,000 tokens
- **VRAM Required**: 20 GB
- **Strengths**: Good performance/resource ratio
- **Best For**: General coding tasks
- **Quantization Options**: Q4_K_M (18GB), Q5_K_M (22GB)

#### qwen2.5-coder-14b
- **Context Window**: 131,072 tokens
- **VRAM Required**: 10 GB
- **Strengths**: Very large context in small footprint
- **Best For**: Resource-constrained environments
- **Quantization Options**: Q4_K_M (8GB), Q8_0 (15GB)

#### starcoder2-15b
- **Context Window**: 16,384 tokens
- **VRAM Required**: 10 GB
- **Strengths**: Fast code completion
- **Best For**: IDE-style completions, quick edits
- **Note**: Smaller context but very fast inference

### Tier 3 - Minimal Resources

#### deepseek-coder-6.7b
- **Context Window**: 16,384 tokens
- **VRAM Required**: 5 GB
- **Strengths**: Good coding in small package
- **Best For**: Quick fixes, single-file tasks

#### codellama-7b
- **Context Window**: 16,384 tokens
- **VRAM Required**: 5 GB
- **Strengths**: Baseline Meta quality
- **Best For**: Basic coding assistance

#### magicoder-7b
- **Context Window**: 16,384 tokens
- **VRAM Required**: 5 GB
- **Strengths**: OSS-Instruct training
- **Best For**: Following coding instructions

---

## Reasoning Specialists

### Tier 1 - Best Performance

#### qwen3-235b
- **Context Window**: 131,072 tokens
- **VRAM Required**: 140 GB (multi-GPU)
- **Strengths**: Near-frontier reasoning
- **Best For**: Complex architecture decisions, algorithm design
- **Note**: Requires significant hardware

#### deepseek-r1
- **Context Window**: 128,000 tokens
- **VRAM Required**: 48 GB
- **Strengths**: Chain-of-thought reasoning
- **Best For**: Step-by-step problem solving
- **Note**: Shows reasoning process

#### llama-4-maverick
- **Context Window**: 128,000 tokens
- **VRAM Required**: 48 GB
- **Strengths**: Creative problem solving
- **Best For**: Novel solutions, brainstorming

#### qwen2.5-72b
- **Context Window**: 131,072 tokens
- **VRAM Required**: 48 GB
- **Strengths**: Strong general reasoning
- **Best For**: Technical decisions, analysis

### Tier 2 - Good Balance

#### llama-3.3-70b
- **Context Window**: 128,000 tokens
- **VRAM Required**: 40 GB
- **Strengths**: Balanced reasoning and coding
- **Best For**: General technical tasks

#### mixtral-8x22b
- **Context Window**: 65,536 tokens
- **VRAM Required**: 48 GB (MoE)
- **Strengths**: Diverse expertise via MoE
- **Best For**: Multi-domain problems

#### qwen2.5-32b
- **Context Window**: 131,072 tokens
- **VRAM Required**: 20 GB
- **Strengths**: Good reasoning in smaller package
- **Best For**: Mid-tier hardware

### Tier 3 - Minimal Resources

#### llama-3.1-8b
- **Context Window**: 128,000 tokens
- **VRAM Required**: 6 GB
- **Strengths**: Large context in small model
- **Best For**: Basic reasoning tasks

---

## Analysis Specialists

> **Note**: Analysis tasks REQUIRE Serena MCP for semantic code understanding

### Recommended for Code Review

| Model | VRAM | Best For |
|-------|------|----------|
| qwen3-coder-30b | 20 GB | Security audits, deep analysis |
| deepseek-coder-v2 | 48 GB | Complex vulnerability detection |
| codellama-34b-instruct | 20 GB | General code review |

### Analysis Workflow

1. **Serena gathers**: symbols, references, diagnostics
2. **Model analyzes**: patterns, vulnerabilities, improvements
3. **Serena applies**: suggested fixes

---

## Documentation Specialists

### Tier 1 - Best Writing

#### qwen2.5-72b
- **VRAM**: 48 GB
- **Strengths**: Clear technical writing
- **Best For**: API docs, technical specs

#### llama-3.3-70b
- **VRAM**: 40 GB
- **Strengths**: Natural language quality
- **Best For**: User guides, READMEs

### Tier 2 - Good Balance

#### mistral-nemo-12b
- **VRAM**: 8 GB
- **Strengths**: Efficient documentation
- **Best For**: Quick docs, comments

#### qwen2.5-32b
- **VRAM**: 20 GB
- **Strengths**: Good quality/size ratio
- **Best For**: Mid-tier documentation

### Tier 3 - Minimal Resources

#### mistral-7b
- **VRAM**: 5 GB
- **Strengths**: Basic documentation
- **Best For**: Simple comments, basic docs

---

## Model Family Characteristics

### Qwen Family
- **Tokenizer**: Custom (slightly higher token count)
- **Context**: Typically 128K+
- **Strengths**: Large context windows, multilingual
- **Note**: Excellent for code across languages

### Llama Family
- **Tokenizer**: SentencePiece
- **Context**: Up to 128K (Llama 3.x)
- **Strengths**: Well-documented, broad ecosystem
- **Note**: Many fine-tuned variants available

### DeepSeek Family
- **Tokenizer**: tiktoken-compatible
- **Context**: 128K
- **Strengths**: Strong coding and reasoning
- **Note**: MoE variants very efficient

### Mistral Family
- **Tokenizer**: SentencePiece
- **Context**: 32K-128K
- **Strengths**: Fast inference, good quality
- **Note**: Mixtral MoE variants available

---

## Quantization Guide

### Quantization Levels

| Level | Quality Loss | VRAM Reduction | Use Case |
|-------|--------------|----------------|----------|
| Q8_0 | ~1% | 50% | Best quality |
| Q6_K | ~2% | 60% | Good balance |
| Q5_K_M | ~3% | 65% | Recommended |
| Q4_K_M | ~5% | 70% | VRAM constrained |
| Q3_K_M | ~10% | 75% | Extreme constraint |

### Recommended Quantizations

| Model | VRAM Available | Quantization |
|-------|----------------|--------------|
| codellama-70b | 48 GB | Q4_K_M |
| codellama-70b | 80 GB | Q8_0 |
| qwen2.5-72b | 48 GB | Q4_K_M |
| qwen2.5-32b | 24 GB | Q5_K_M |
| qwen2.5-32b | 16 GB | Q4_K_M |

---

## Hardware Recommendations

### Minimal Setup (5-8 GB VRAM)
- **GPU**: RTX 3060 12GB, RTX 4060 8GB
- **Models**: 7B models, 13B Q4 quantized
- **Tasks**: Basic coding, simple docs

### Standard Setup (20-24 GB VRAM)
- **GPU**: RTX 3090, RTX 4090, A5000
- **Models**: 30-34B models, 70B Q4 quantized
- **Tasks**: Full coding, analysis, docs

### Professional Setup (48+ GB VRAM)
- **GPU**: A6000, A100, 2x RTX 4090
- **Models**: 70B+ full precision, MoE models
- **Tasks**: Complex reasoning, deep analysis

### Enterprise Setup (Multi-GPU)
- **GPU**: 4x A100, 8x H100
- **Models**: 200B+ models
- **Tasks**: Frontier-level capabilities

---

## Service-Specific Notes

### Ollama
- Native API (not OpenAI-compatible)
- Model names: `ollama pull qwen2.5-coder:14b`
- Automatic quantization selection

### LM Studio
- OpenAI-compatible API
- GGUF format models
- Good GUI for model management

### Jan
- OpenAI-compatible API
- Built-in model downloader
- Cross-platform desktop app

### LocalAI
- Full OpenAI API compatibility
- Supports multiple backends
- Good for API-first deployments
