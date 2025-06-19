# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FinGPT is an open-source Financial Large Language Model project that democratizes financial AI. It provides pre-trained and fine-tunable models specifically for financial tasks including sentiment analysis, stock forecasting, NER, and report analysis.

## Key Architecture

### Module Structure
```
fingpt/
├── FinGPT_Benchmark/          # Evaluation suite with standardized datasets
├── FinGPT_Forecaster/         # Stock prediction using news & market data
├── FinGPT_RAG/               # Retrieval Augmented Generation implementation
├── FinGPT_Sentiment_Analysis_v1-v3/  # Progressive versions of sentiment models
├── FinGPT_FinancialReportAnalysis/   # SEC filing & report analysis
├── FinGPT_MultiAgentsRAG/     # Multi-agent financial reasoning system
└── FinGPT_Others/            # Trading strategies, Robo-advisor modules
```

### Core Components
- **Base Models**: Supports Llama2, ChatGLM2, Falcon, MPT, Bloomz, Qwen
- **Training**: Uses LoRA/QLoRA for parameter-efficient fine-tuning via PEFT
- **Distributed Training**: DeepSpeed integration for multi-GPU setups
- **Data Pipeline**: Custom loaders for financial datasets (news, reports, market data)

## Common Commands

### Training Models
```bash
# LoRA fine-tuning with DeepSpeed
deepspeed train_lora.py \
  --base_model llama2 \
  --dataset sentiment-train \
  --max_length 512 \
  --batch_size 4 \
  --learning_rate 1e-4 \
  --num_epochs 8

# QLoRA (4-bit quantization) for memory efficiency
python train_qlora.py \
  --base_model llama2-7b \
  --dataset_name fingpt-sentiment-train \
  --max_length 512 \
  --per_device_train_batch_size 1
```

### Evaluation & Benchmarking
```bash
# Run benchmarks on specific task
python benchmarks.py \
  --dataset re \
  --base_model llama2 \
  --peft_model ../finetuned_models/[model_name] \
  --batch_size 8

# Evaluate sentiment model
python test_sentiment.py \
  --base_model llama2-13b \
  --peft_model FinGPT/fingpt-sentiment_llama2-13b_lora
```

### Running Inference
```bash
# Interactive demo
python demo.py \
  --base_model llama2-7b \
  --peft_model [path_to_finetuned_model] \
  --test_dataset prompt

# Batch prediction
python predict.py \
  --input_file data/test.csv \
  --output_file predictions.csv \
  --model_path [model_path]
```

## Development Workflow

### Adding New Financial Tasks
1. Create dataset loader in `fingpt/FinGPT_Benchmark/datasets/`
2. Implement task-specific prompt templates
3. Add evaluation metrics to benchmarking suite
4. Update training scripts with task configuration

### Model Integration Pattern
```python
# Standard pattern for new model integration
from transformers import AutoTokenizer, AutoModelForCausalLM
from peft import PeftModel

base_model = AutoModelForCausalLM.from_pretrained(base_model_path)
peft_model = PeftModel.from_pretrained(base_model, peft_model_path)
tokenizer = AutoTokenizer.from_pretrained(base_model_path)
```

### Data Processing Pipeline
- Financial news: Processed through `fingpt/FinGPT_Forecaster/data_preparations/`
- Market data: Integrated via yfinance/tushare APIs
- Reports: PDF parsing and chunking in `FinGPT_FinancialReportAnalysis/`

## Testing

```bash
# Run unit tests for specific module
cd fingpt/FinGPT_Sentiment_Analysis_v3
python -m pytest tests/

# Validate model outputs
python validate_outputs.py --task sentiment --model [model_path]
```

## Important Considerations

### Memory Management
- Use QLoRA for models > 7B parameters on consumer GPUs
- Enable gradient checkpointing for large batch training
- Monitor VRAM usage with `nvidia-smi`

### Financial Data Handling
- Always validate market data timestamps and alignment
- Handle missing/null values in financial time series
- Respect rate limits on financial APIs (yfinance, finnhub)

### Model Selection
- Sentiment: Use v3.5 models for best accuracy
- Forecasting: Llama2-based models perform best
- Chinese markets: Use ChatGLM2 or Qwen base models

## Key Dependencies
- PyTorch 2.0.1
- Transformers 4.32.0-4.34.0
- PEFT 0.5.0
- DeepSpeed 0.10.0+
- Python 3.6+