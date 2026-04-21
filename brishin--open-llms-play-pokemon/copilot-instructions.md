## open-llms-play-pokemon

> When using dsy, the LLM/AI framework

# DSPy: Programming Foundation Models

## What is DSPy?

DSPy is a framework for programming—rather than prompting—language models. It allows you to build modular AI systems and offers algorithms for optimizing their prompts and weights. Instead of brittle prompts, you write compositional Python code and use DSPy to teach your LM to deliver high-quality outputs.

**Key Value Proposition:** DSPy replaces manual prompt engineering with automated optimization, making AI systems more reliable, maintainable, and effective.

## Installation

```bash
# Install DSPy
pip install dspy

# Install from latest main branch
pip install git+https://github.com/stanfordnlp/dspy.git

# With optional dependencies
pip install dspy[anthropic,aws,mcp]  # For specific providers
```

## Core Concepts

### 1. Signatures
Define the input/output structure of your AI modules:

```python
import dspy

class TranslateSignature(dspy.Signature):
    """Translate text from one language to another."""
    text = dspy.InputField()
    source_language = dspy.InputField()
    target_language = dspy.InputField()
    translation = dspy.OutputField()

# Or use string syntax
translate_sig = dspy.Signature("text, source_language, target_language -> translation")
```

### 2. Modules
Building blocks that use language models:

```python
class Translator(dspy.Module):
    def __init__(self):
        super().__init__()
        self.translate = dspy.Predict(TranslateSignature)
    
    def forward(self, text, source_language, target_language):
        return self.translate(
            text=text,
            source_language=source_language,
            target_language=target_language
        ).translation
```

### 3. Language Model Configuration
Configure any supported LM provider:

```python
# OpenAI
lm = dspy.LM(model="openai/gpt-4o-mini")

# Anthropic Claude
lm = dspy.LM(model="anthropic/claude-3-5-sonnet-20241022")

# Local models via Ollama
lm = dspy.LM(model="ollama/llama3.2")

# Configure DSPy to use the model
dspy.settings.configure(lm=lm)
```

### 4. Built-in Modules
DSPy provides powerful pre-built modules:

```python
# Chain of Thought reasoning
cot = dspy.ChainOfThought(TranslateSignature)

# Retrieve and generate (RAG)
class RAGModule(dspy.Module):
    def __init__(self, retriever):
        super().__init__()
        self.retrieve = retriever
        self.generate = dspy.ChainOfThought("question, context -> answer")
    
    def forward(self, question):
        context = self.retrieve(question, k=3)
        return self.generate(question=question, context=context)

# Multi-step reasoning
class MultiStepReasoner(dspy.Module):
    def __init__(self):
        super().__init__()
        self.analyze = dspy.ChainOfThought("problem -> analysis, sub_problems")
        self.solve = dspy.ChainOfThought("sub_problem -> solution")
        self.synthesize = dspy.ChainOfThought("solutions -> final_answer")
    
    def forward(self, problem):
        analysis = self.analyze(problem=problem)
        solutions = [self.solve(sub_problem=sp) for sp in analysis.sub_problems]
        return self.synthesize(solutions=solutions)
```

## Optimization (The Magic of DSPy)

DSPy's real power comes from 14 different automatic optimization algorithms:

### Few-Shot Learning Optimizers
```python
# Create training examples
trainset = [
    dspy.Example(
        text="Hello world",
        source_language="English", 
        target_language="Spanish",
        translation="Hola mundo"
    ).with_inputs("text", "source_language", "target_language"),
    # Add more examples...
]

# Bootstrap Few-Shot Learning
bootstrap = dspy.BootstrapFewShot(max_bootstrapped_demos=8)
optimized = bootstrap.compile(student=Translator(), trainset=trainset)

# Bootstrap with Random Search (explores multiple program candidates)
bootstrap_rs = dspy.BootstrapRS(max_bootstrapped_demos=4, max_labeled_demos=16)
optimized = bootstrap_rs.compile(student=module, trainset=trainset)

# K-Nearest Neighbor Few-Shot (dynamic demo selection)
knn = dspy.KNNFewShot(k=3, trainset=trainset)
optimized = knn.compile(student=module)
```

### Instruction Optimization
```python
# COPRO: Coordinate Prompt Optimization
copro = dspy.COPRO(metric=accuracy, breadth=10, depth=3)
optimized = copro.compile(student=module, trainset=trainset)

# Auto-generates better instructions based on performance feedback
```

### State-of-the-Art Multi-Modal Optimization
```python
# MIPROv2: Multi-prompt Instruction Proposal Optimizer
mipro = dspy.MIPROv2(
    metric=accuracy,
    num_candidates=25,       # Number of instruction candidates
    init_temperature=1.4,    # Creativity in instruction generation
    verbose=True
)
optimized = mipro.compile(
    student=module, 
    trainset=trainset,
    num_trials=100,         # Bayesian optimization trials
    max_errors=5,
    requires_permission_to_run=False
)

# SIMBA: Stochastic Introspective Mini-Batch Ascent
simba = dspy.SIMBA(
    metric=accuracy,
    num_candidates=7,
    max_num_traces=3       # Multiple execution traces per example
)
optimized = simba.compile(student=module, trainset=trainset)
```

### Fine-Tuning and Weight Optimization
```python
# Fine-tune model weights
finetune = dspy.BootstrapFinetune(
    metric=accuracy,
    target='openai/gpt-3.5-turbo',
    epochs=1,
    multitask=False
)
optimized = finetune.compile(student=module, trainset=trainset)

# GRPO: Reinforcement Learning-based optimization
grpo = dspy.GRPO(
    metric=accuracy,
    target_lm=target_model,
    max_steps=100
)
optimized = grpo.compile(student=module, trainset=trainset)
```

### Meta and Ensemble Optimizers
```python
# BetterTogether: Sequential prompt + weight optimization
better_together = dspy.BetterTogether(
    prompt_optim=dspy.BootstrapRS(),
    weights_optim=dspy.BootstrapFinetune(),
    strategy="p -> w -> p"  # prompt first, then weights, then prompt again
)
optimized = better_together.compile(student=module, trainset=trainset)

# Ensemble multiple optimized programs
ensemble = dspy.Ensemble(reduce_fn=dspy.majority)
optimized = ensemble.compile([program1, program2, program3])

# Avatar Optimizer for tool-using agents
avatar = dspy.AvatarOptimizer(
    metric=accuracy,
    max_iters=3,
    lower_bound=0.0
)
optimized = avatar.compile(student=agent_module, trainset=trainset)
```

### Rule-Based and Knowledge Integration
```python
# Infer natural language rules from examples
infer_rules = dspy.InferRules(
    metric=accuracy,
    program_aware=True,
    data_aware=True
)
optimized = infer_rules.compile(student=module, trainset=trainset)
```

### Hyperparameter Optimization
```python
# Bayesian optimization with Optuna
optuna_optimizer = dspy.BootstrapFewShotWithOptuna(
    metric=accuracy,
    num_candidates=20,
    num_threads=4
)
optimized = optuna_optimizer.compile(student=module, trainset=trainset)
```

## Supported Language Models

DSPy supports 50+ LM providers through LiteLLM integration:

### Major Providers
- **OpenAI**: GPT-4o, GPT-4o-mini, O1, GPT-3.5-turbo
- **Anthropic**: Claude-3.5-sonnet, Claude-3-haiku, Claude-3-opus
- **Google**: Gemini-1.5-pro, Gemini-1.5-flash, PaLM
- **Meta**: Llama-3.1, Llama-3.2, Code-Llama
- **Mistral**: Mistral-large, Mistral-7b, Codestral
- **Cohere**: Command-R, Command-R+
- **xAI**: Grok models

### Cloud Platforms
- **Azure OpenAI**: Microsoft-hosted OpenAI models
- **AWS Bedrock**: Claude, Llama, Titan models
- **Google VertexAI**: Gemini and other models
- **Databricks**: Unity Catalog models

### Inference Platforms
- **Ollama**: Local model serving
- **Together AI**: Open source model hosting
- **Fireworks AI**: Fast inference
- **Groq**: Ultra-fast inference
- **Replicate**: Cloud model hosting
- **Anyscale**: Ray-based serving

### Usage Examples
```python
# OpenAI
lm = dspy.LM("openai/gpt-4o-mini", api_key="your-key")

# Anthropic
lm = dspy.LM("anthropic/claude-3-5-sonnet-20241022", api_key="your-key")

# Local Ollama
lm = dspy.LM("ollama/llama3.2", api_base="http://localhost:11434")

# Azure OpenAI
lm = dspy.LM(
    "azure/gpt-4o-mini",
    api_key="your-key",
    api_base="https://your-resource.openai.azure.com",
    api_version="2024-02-01"
)
```

## Advanced Features

### Adapters and Output Formats
DSPy provides sophisticated adapters for different interaction patterns:

```python
# Chat-based interactions with structured fields
chat_adapter = dspy.ChatAdapter()

# JSON structured outputs with validation
json_adapter = dspy.JSONAdapter()

# Two-step processing for reasoning models (O1, etc.)
two_step_adapter = dspy.TwoStepAdapter(extraction_model=dspy.LM("openai/gpt-4o-mini"))

# Configure adapter globally
dspy.settings.configure(lm=lm, adapter=two_step_adapter)
```

### Assertions and Constraints
```python
import dspy

class ConstrainedQA(dspy.Module):
    def __init__(self):
        super().__init__()
        self.qa = dspy.ChainOfThought("question -> answer")
    
    def forward(self, question):
        answer = self.qa(question=question).answer
        
        # Assert constraints
        dspy.Assert(
            len(answer.split()) <= 50,
            "Answer must be 50 words or less"
        )
        dspy.Assert(
            "sorry" not in answer.lower(),
            "Answer should not contain apologies"
        )
        
        return answer
```

### Tool Use and Function Calling
```python
# Define tools from Python functions
@dspy.Tool
def search_web(query: str) -> str:
    """Search the web for information."""
    # Implementation
    return search_results

@dspy.Tool  
async def send_email(recipient: str, subject: str, body: str) -> bool:
    """Send an email to a recipient."""
    # Async tool implementation
    return success

# Use tools in modules
class AgentModule(dspy.Module):
    def __init__(self):
        super().__init__()
        self.planner = dspy.ChainOfThought("task -> action, tool_args")
        
    def forward(self, task):
        plan = self.planner(task=task)
        if plan.action == "search":
            return search_web(plan.tool_args)
        elif plan.action == "email":
            return send_email(**plan.tool_args)
```

### Model Context Protocol (MCP) Integration
```python
# Connect to MCP servers for advanced tool capabilities
from dspy.utils.mcp import create_tools_from_mcp

# Convert MCP tools to DSPy tools
tools = create_tools_from_mcp("mcp://server-url")
```

### Retrieval-Augmented Generation
DSPy supports 15+ retrieval systems:

```python
# Vector databases
retriever = dspy.Pinecone(index_name="documents")
retriever = dspy.Weaviate(url="http://localhost:8080")
retriever = dspy.Qdrant(url="http://localhost:6333")

# Specialized retrievers
retriever = dspy.ColBERTv2(url="http://server:2017/wiki17_abstracts")
retriever = dspy.VectaraRM(api_key="key", corpus_id="corpus")

class RAGPipeline(dspy.Module):
    def __init__(self, retriever, k=3):
        super().__init__()
        self.retrieve = retriever
        self.generate = dspy.ChainOfThought("question, context -> answer")
        self.k = k
    
    def forward(self, question):
        passages = self.retrieve(question, k=self.k)
        context = "\n".join([p.long_text for p in passages])
        return self.generate(question=question, context=context)
```

### Streaming and Async Support
```python
# Real-time streaming with status updates
@dspy.streamify
def streaming_chat(message):
    return dspy.Predict("message -> response")(message=message)

for chunk in streaming_chat("Hello!"):
    print(chunk.chunk, end="")  # StreamResponse objects

# Async support with context preservation
@dspy.asyncify
async def async_translate(text):
    return await dspy.Predict("text -> translation")(text=text)

result = await async_translate("Hello world")

# Custom streaming listeners
stream_listeners = [dspy.StreamListener("response")]
dspy.settings.configure(stream_listeners=stream_listeners)
```

### Multi-modal Support
```python
# Images with flexible input sources
image = dspy.Image(url="https://example.com/image.jpg")
image = dspy.Image(path="/path/to/image.jpg")  
image = dspy.Image(pil_image=pil_obj)
image = dspy.Image(base64="data:image/jpeg;base64,...")

response = dspy.Predict("image -> description")(image=image)

# Audio processing
audio = dspy.Audio(path="/path/to/audio.wav")
audio = dspy.Audio(url="https://example.com/audio.mp3")
audio = dspy.Audio(numpy_array=audio_data, sample_rate=16000)

# History for conversational AI
history = dspy.History([
    {"role": "user", "content": "Hello"},
    {"role": "assistant", "content": "Hi there!"}
])

class ChatBot(dspy.Module):
    def __init__(self):
        super().__init__()
        self.chat = dspy.Predict("history, message -> response")
    
    def forward(self, message, history):
        return self.chat(history=history, message=message)
```

### Python Code Execution
```python
# Sandboxed Python interpreter using Deno + Pyodide
interpreter = dspy.PythonInterpreter()

code = """
import numpy as np
data = np.array([1, 2, 3, 4, 5])
result = np.mean(data)
"""

# Execute with variable injection
result = interpreter.run(code, inject_vars={"external_data": [1,2,3]})
print(result.variables["result"])  # 3.0
```

## Evaluation and Metrics

DSPy provides comprehensive evaluation capabilities with parallel processing and advanced metrics:

### Core Evaluation Framework
```python
from dspy.evaluate import Evaluate

# Built-in exact match metrics
def accuracy(gold, pred, trace=None):
    return gold.answer == pred.answer

# Multi-threaded evaluation with progress tracking
evaluator = Evaluate(
    devset=test_examples,
    metric=accuracy,
    num_threads=8,              # Parallel evaluation
    display_progress=True,      # Real-time progress bar
    display_table=10,           # Show top 10 results
    max_errors=5,              # Error tolerance
    return_all_scores=True,     # Individual example scores
    return_outputs=True         # Full prediction details
)

# Get comprehensive results
score = evaluator(optimized_module)
```

### Built-in Metrics
```python
# Exact matching with variants
from dspy.evaluate import answer_exact_match, answer_passage_match

# Text processing utilities
from dspy.evaluate import normalize_text, f1_score

# Advanced semantic metrics
from dspy.evaluate import EM, F1, HotPotF1

# Multi-criteria evaluation
def comprehensive_metric(gold, pred, trace=None):
    exact_match = answer_exact_match(gold, pred)
    passage_support = answer_passage_match(gold, pred) 
    return exact_match and passage_support
```

### LM-based Evaluation (Auto-Evaluation)
```python
from dspy.evaluate import SemanticF1, CompleteAndGrounded

# Semantic similarity evaluation using LMs
semantic_eval = SemanticF1(
    threshold=0.75,            # Similarity threshold
    decompositional=True       # Detailed reasoning
)

# Dual evaluation for completeness and groundedness  
quality_eval = CompleteAndGrounded(
    threshold=0.8,
    completeness_model=dspy.LM("openai/gpt-4o-mini"),
    groundedness_model=dspy.LM("openai/gpt-4o-mini")
)

# Use in evaluation
evaluator = Evaluate(devset=devset, metric=semantic_eval)
score = evaluator(module)
```

### Custom Metrics and Complex Evaluation
```python
def multi_hop_validation(example, pred, trace=None):
    """Validate multi-hop reasoning with trace analysis."""
    # Check answer accuracy
    if not answer_exact_match(example, pred):
        return False
        
    # Validate reasoning chain
    if trace:
        hops = [example.question] + [
            outputs.query for *_, outputs in trace 
            if hasattr(outputs, "query")
        ]
        # Check hop quality and diversity
        if any(len(h) > 100 for h in hops):
            return False
        if len(set(hops)) != len(hops):  # Check uniqueness
            return False
    
    return True

# Use in evaluation
evaluator = Evaluate(
    devset=complex_qa_dataset,
    metric=multi_hop_validation,
    num_threads=4,
    provide_traceback=True  # Include full traces
)
```

### Data Loading and Management
```python
# Built-in datasets
from dspy.datasets import GSM8K, HotPotQA, MATH, Colors

# Load with configurable splits
gsm8k = GSM8K()
hotpot = HotPotQA(train_size=100, eval_size=200, train_seed=42)
math_data = MATH("algebra")  # Specific subset

# Multi-format data loading
from dspy.datasets import DataLoader
loader = DataLoader()

# From various sources
data = loader.from_huggingface("squad", split="train")
data = loader.from_csv("data.csv", input_keys=["question"])
data = loader.from_json("data.jsonl", fields=["text", "label"])
data = loader.from_pandas(df, input_keys=["input"])

# Data splitting and sampling
train, test = loader.train_test_split(data, train_size=0.8)
sample = loader.sample(data, n=100)

# Create Examples programmatically
examples = [
    dspy.Example(
        question=row["question"],
        answer=row["answer"]
    ).with_inputs("question")
    for row in data
]
```

## Production Features

### Multi-level Caching System
```python
# Configure sophisticated caching
dspy.configure_cache(
    enable_disk_cache=True,           # Persistent FanoutCache on disk
    enable_memory_cache=True,         # In-memory LRU cache
    disk_cache_dir="~/.dspy_cache",
    disk_size_limit_bytes=30_000_000_000,  # 30GB
    memory_max_entries=1000000,       # 1M entries in memory
    enable_litellm_cache=False        # Use DSPy cache instead
)

# Thread-safe operations with JSON serialization
# Automatic cache key generation for complex objects
```

### Usage Tracking and Cost Management
```python
# Comprehensive token and cost tracking
with dspy.track_usage() as usage:
    result = module(input_data)

print(f"Total tokens: {usage.total_tokens}")
print(f"Prompt tokens: {usage.prompt_tokens}")
print(f"Completion tokens: {usage.completion_tokens}")
print(f"Total cost: ${usage.total_cost}")

# Model-specific breakdowns
for model, stats in usage.model_usage.items():
    print(f"{model}: {stats.total_tokens} tokens, ${stats.total_cost}")
```

### Saving and Loading with Rich Metadata
```python
# Save optimized modules with full state
module.save("optimized_model.json")

# Load preserves all optimization state
loaded_module = dspy.load("optimized_model.json")

# Save/load entire program state including settings
dspy.save_state("full_program.json")
dspy.load_state("full_program.json")
```

### Fine-tuning and Model Management
```python
# Fine-tune models for production
finetune_job = dspy.BootstrapFinetune(
    target="openai/gpt-3.5-turbo-1106",
    multitask=True,
    epochs=3
)

# Track training jobs
job_id = finetune_job.train(trainset)
status = finetune_job.get_status(job_id)

# Use fine-tuned models
optimized_lm = dspy.LM(model=f"ft:{job_id}")
```

### Callbacks and Monitoring
```python
from dspy.utils.callback import BaseCallback

class ProductionMonitor(BaseCallback):
    def on_module_start(self, module, args, kwargs):
        # Log module execution
        pass
    
    def on_lm_end(self, lm, prompt, response, metadata):
        # Monitor LM usage
        pass
    
    def on_error(self, error, module=None):
        # Handle errors gracefully
        pass

# Configure global monitoring
dspy.settings.configure(
    lm=lm,
    callbacks=[ProductionMonitor()]
)
```

### Context Management and Thread Safety
```python
# Thread-safe configuration management
with dspy.settings.context(
    lm=dspy.LM("openai/gpt-4o"),
    async_max_workers=16
):
    # Settings isolated to this context
    result = module(input_data)

# Async execution with proper context propagation
@dspy.asyncify
async def async_pipeline(inputs):
    # Context preserved across async boundaries
    return [await module.acall(inp) for inp in inputs]
```

### Experimental and Research Features
```python
# Synthetic data generation for training augmentation
from dspy.experimental import SyntheticDataGenerator

generator = SyntheticDataGenerator(
    schema_class=MyDataSchema,
    examples=seed_examples
)
synthetic_data = generator.generate(n=1000)

# Module graph visualization for debugging
from dspy.experimental import ModuleGraph

graph = ModuleGraph(my_complex_program)
graph.visualize("program_flow.png")

# Advanced optimization with feedback
from dspy.experimental.synthesizer import Synthesizer

synthesizer = Synthesizer(
    input_lm=dspy.LM("openai/gpt-4o"),
    output_lm=dspy.LM("openai/gpt-4o-mini")
)
improved_examples = synthesizer.synthesize(task_description, n=100)
```

## Best Practices

1. **Start Simple**: Begin with basic signatures and `dspy.Predict` modules
2. **Use Examples**: Provide high-quality training examples with proper `.with_inputs()` specification
3. **Choose the Right Optimizer**: 
   - `BootstrapFewShot` for basic optimization
   - `MIPROv2` for state-of-the-art instruction + demo optimization  
   - `BootstrapFinetune` for model weight optimization
   - `GRPO` for reinforcement learning scenarios
4. **Evaluate Systematically**: Use `dspy.Evaluate` with proper metrics and parallel processing
5. **Cache Intelligently**: Enable multi-level caching for development efficiency
6. **Use Assertions**: Implement `dspy.Assert` for output quality constraints
7. **Modular Design**: Break complex tasks into composable modules
8. **Leverage Adapters**: Use `TwoStepAdapter` for reasoning models, `JSONAdapter` for structured outputs
9. **Monitor Production**: Implement callbacks for usage tracking and error handling
10. **Optimize Iteratively**: Start with few-shot → instruction optimization → fine-tuning

## Community and Resources

- **Documentation**: https://dspy.ai
- **GitHub**: https://github.com/stanfordnlp/dspy
- **Discord**: https://discord.gg/XCGy2WDCQB
- **Papers**: Key research papers available in README

## Getting Started

1. **Install DSPy**: `pip install dspy`
2. **Configure your LM**: `dspy.settings.configure(lm=dspy.LM("openai/gpt-4o-mini"))`
3. **Define a signature**: `sig = dspy.Signature("input -> output")`
4. **Create a module**: `module = dspy.Predict(sig)`
5. **Use it**: `result = module(input="Hello")`
6. **Create training data**: Examples with `.with_inputs()` specification
7. **Optimize it**: Use teleprompters (`BootstrapFewShot`, `MIPROv2`, etc.)
8. **Evaluate performance**: `dspy.Evaluate` with custom metrics
9. **Deploy it**: Save optimized modules and enable production monitoring

## Key Advantages of DSPy

- **🔄 Automated Optimization**: 14 different optimization algorithms replace manual prompt engineering
- **📊 Systematic Evaluation**: Built-in parallel evaluation with custom metrics
- **🛠️ Production Ready**: Caching, monitoring, callbacks, and fine-tuning support
- **🔧 Modular Design**: Composable modules with clean abstractions
- **🌐 Universal Compatibility**: 50+ LM providers through unified interface
- **⚡ Performance**: Streaming, async, and parallel execution support
- **🔍 Introspection**: Rich debugging, tracing, and visualization tools

DSPy transforms how you build with language models—from brittle prompting to reliable, optimizable programming.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brishin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
