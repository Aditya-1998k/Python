### t-string
Like f-strings, but allow transformations on the string automatically.
Not yet standard — still in discussion/PEP stage.
```python
f"Hello {name}"     # f-string → runtime interpolation
t"SELECT * FROM {table}"  # t-string → transformable at parse/compile time
```

Potential usages:
1. Auto-escaping SQL queries
2. Safer regex building
3. Custom templating languages

### DSPy (Declarative Self-improving Python)
Documantation: [DSPy](https://dspy.ai/)
A framework for building LLM pipelines that self-optimize.
Instead of brittle prompts, you declare what you want, and DSPy learns how.

```python
import dsp

# Define a pipeline
class Summarizer(dsp.Module):
    def __init__(self):
        self.summarize = dsp.Summarize(max_words=20)

    def forward(self, doc: str) -> str:
        return self.summarize(doc)

# Run pipeline (DSPy optimizes prompts internally)
summ = Summarizer()
print(summ("Python is an interpreted high-level programming language..."))
```
Output:
```
Short, optimized summary (DSPy ensures constraint).
```

Usage:
1. Works with OpenAI, HuggingFace, local LLMs.
2. Can do RAG (retrieval-augmented generation).
3. Learns better prompts automatically (“prompt compiler”).
4. Great for structured output (JSON, tables, etc.).
