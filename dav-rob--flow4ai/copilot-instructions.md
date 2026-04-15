## flow4ai

> Every time you choose to apply a rule(s), explicitly state the rule(s) in the output. You can abbreviate the rule description to a single word or phrase.

# Flow4AI - scalable AI job scheduling and execution platform

Every time you choose to apply a rule(s), explicitly state the rule(s) in the output. You can abbreviate the rule description to a single word or phrase.

## Project Context
Flow4AI allows users to execute AI and IO Job graphs in parallel.

## Working process


## Architecture
Always read the [architecture document](docs/ARCHITECTURE.md) before undertaking any task. You are allowed to update this document with any observations you have made. If you do update the architecture.md, then put *** ARCHITECTURE UPDATE *** in the output.

## Tests
- execute unit tests in the root directory using pytest using the command `python -m pytest` as the root of your test commands.
- use pytest's --full-suite flag to run all tests, including performance tests.
- when running tests that are logging in order to investigate causes  use `python -m pytest -s` or `python -m pytest my_test_file.py -s` to see the logs. Just using "-v" will not show the logs.

## JobABC Implementation Rules
- NEVER override the `_execute` method in custom job classes. This method is responsible for job graph execution and state management.
- Always implement the required `run` method in custom job classes to define job-specific behavior.
- The `run` method should accept a task parameter and return a dictionary with results.

## Debugging FlowManagerMP Timeout Errors
When encountering "Timed out waiting for jobs to be loaded" errors in FlowManagerMP:
- Check the stderr output from the JobExecutorProcess for underlying errors

## Logging
For logging, use the flow4ai.f4a_logging module, not the standard Python logging module.
- for external files, examples and tests use
`import flow4ai.f4a_logging as logging` 
then do 
`logger = logging.getLogger(__name__)`
- for internal files under /src use the relative path import of f4a_logging
- to activate debug logging in tests do `FLOW4AI_LOG_LEVEL=DEBUG python -m pytest -v -s`

## Core Concepts

### Flow4AI
A sophisticated Python framework designed for parallel and asynchronous job execution, with a focus on AI and data processing workflows. Flow4AI provides a flexible, graph-based job scheduling system that enables complex task dependencies and parallel processing.

### Job Graph
The complete workflow of interconnected jobs that defines the execution flow. Job graphs represent the directed acyclic graph (DAG) of execution units where each node is a `JobABC` instance, often wrapping functions from LangChain, LlamaIndex, or other AI and data processing frameworks. While "workflow" describes the higher-level business process being automated, "job graph" specifically refers to the technical implementation of execution flow.

### DSL (Domain Specific Language)
A Python-based interface for defining workflow configurations programmatically. The DSL allows developers to construct job graphs using intuitive syntax and object-oriented patterns.

### Workflow Configuration
A declarative representation of job graphs using YAML or JSON files. These configurations provide an alternative to the DSL approach, allowing workflows to be defined, stored, and version-controlled in a human-readable format.

### Job
A single execution unit in the workflow that inherits from `JobABC`. Each job takes inputs from preceding jobs and passes its output to subsequent jobs in the job graph. Jobs encapsulate discrete processing logic and maintain isolation between task executions.

### Head Job
The entry point or starting job in a job graph. If multiple starting jobs are defined, a `DefaultHeadJob` is automatically added to serve as a single entry point. The name of the entire job graph equals the name of the head job.

### Tail Job
The final job in a job graph that produces the ultimate result. This job's output is returned by default as the final result of workflow execution. If multiple endpoints exist in the graph, a default tail job is created to consolidate results from all terminal jobs.

### Short Job Name
The simplified identifier used for referencing jobs within a workflow. This is the name assigned to a job in the `wrap()` function when using the DSL, or in the job definition when using YAML configuration files.

### FQ_Name (Fully Qualified Name)
An expanded job name created by the workflow engine to prevent naming collisions across job graphs. The format follows `graph_name$$param_name$$job_name$$`, providing a unique identifier for each job in the system.  Indices are added to the param_name to prevent collisions.

## Operational Components

### Task
A dictionary subclass with a unique identifier that represents a unit of work to be processed by a job graph. Tasks contain input parameters and maintain metadata throughout execution. Tasks are units of work (data + metadata) that flow through the job graph, while jobs are the processing units that operate on tasks. This distinction is fundamental to understanding the Flow4AI execution model.

### FlowManager
A singleton class that manages job graphs and tasks. The FlowManager maintains internal state about job graphs, tasks, and results, and provides methods for submission, waiting, and result collection.

### FlowManagerMP
A multiprocessing implementation of FlowManager that enables parallel task execution across multiple processes.

### JobABC
The abstract base class that defines the core contract for job execution. All custom jobs must inherit from this class and implement the required `run` method.

### WrappingJob
A specialized job implementation that wraps regular Python functions, enabling them to be integrated into Flow4AI job graphs without creating custom `JobABC` subclasses.

## Technical Patterns

### Job Context (j_ctx)
A special parameter used in wrapped functions to access task data, inputs from predecessor jobs, and other contextual information required for execution.

### get_inputs()
A method available in `JobABC` subclasses that provides access to inputs from predecessor jobs, returning a dictionary with short job names as keys.

## Recommended Practices

### Job Execution Paradigms: JobABC Subclasses vs. Wrapped Functions

Flow4AI supports two primary approaches for defining jobs, each with distinct use cases:

#### When to Use Wrapped Functions

- **Framework Integration**: Wrapped functions were designed to allow seamless integration with diverse frameworks like LangChain, LlamaIndex, or any other AI or data processing framework that users are already familiar with.

- **Simplicity and Familiarity**: For users who prefer writing standard Python functions, wrapped functions provide a straightforward approach with no reduction in functionality compared to JobABC subclasses.

- **Minimal Dependencies**: When a job doesn't need to access inputs from predecessor jobs or task metadata, wrapped functions without the `j_ctx` parameter offer the cleanest implementation.

#### When to Use JobABC Subclasses

- **Built-in Context Access**: JobABC subclasses provide built-in `get_inputs()` and `get_task()` methods, making access to predecessor outputs and task data more direct without needing to extract this information from a context parameter.

- **Object-Oriented Design**: For complex jobs that benefit from encapsulation, inheritance, and other OOP principles, subclassing JobABC provides a more natural structure.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dav-rob) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
