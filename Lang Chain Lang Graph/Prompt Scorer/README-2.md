# 🔮 Prompt Quality Scoring Agent (Colab)


## Quick Start

1. Pick a version above and click **Open in Colab**
2. Set API key in Colab secrets (left sidebar → 🔑 Secrets)
3. Run all cells (Runtime → Run all)

## Import in Your Own Notebook

```python
# Gemini version
!pip install -q langchain langchain-google-genai
!curl -sO https://raw.githubusercontent.com/rdpgpuvm/prompt-scorer-colab/main/colab_import.py
from colab_import import score_prompt, print_score

# OpenAI version
!pip install -q langchain langchain-openai
!curl -sO https://raw.githubusercontent.com/rdpgpuvm/prompt-scorer-colab/main/openai_scorer.py
from openai_scorer import score_prompt, print_score
```

