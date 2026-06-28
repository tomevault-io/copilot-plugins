## verbalization

> **Important.** before running any code or pytest you need to source the `venv` at the following location:

## Development Instructions

**Important.** before running any code or pytest you need to source the `venv` at the following location:

```bash
source python/verbalization/.venv/bin/activate
```

## Project Organization

This is an ArmarX skills package that combines episodic memory and LLMs for robot experience verbalization. The project has both C++ ArmarX components and Python packages.

### C++ Source Code

- `source/armarx/verbalization/`:
  - `skills/skills/EpisodicVerbalization.{h,cpp}`: C++ implementation of the EpisodicVerbalization skill
  - `components/episodic_verbalization_skill_provider/Component.{h,cpp}`: ArmarX component that provides the verbalization skill

### Python Core (Legacy/Experimental)

- `python/core/`:
  - `tests/`: Test scripts and experimental code
    - `matching_test.py`: Tests for question matching logic
    - `manual_test.py`: Manual testing script
    - `experiments/`: Experimental scripts and evaluation
      - `Experiment.py`: Experiment framework
      - `automatic_experiment.py`: Automated experiment runner
      - `scripts/`: Various experiment scripts organized by date and type
        - `Humanoids2025_experiments/`: Specific question type experiments
        - `by date/`: Scripts organized chronologically
      - `data/`: Experiment data
      - `logs/`: Experiment logs
    - `data/`: Test data files (JSON exports for various question types)
  - `verbalizing/ManualQuestionDefinitions/`: Legacy manual question definitions (older versions)

### Python Verbalization Package (Main Implementation)

- `python/verbalization/verbalization/`: Main verbalization package
  - **Core Modules:**
    - `services/`: Service implementations
      - `episodic_verbalization_service.py`: Main episodic verbalization service
      - `llm_verbalization_service.py`: LLM-based verbalization service
    - `verbalizing/`: Core verbalization logic
      - `AnswerGenerator.py`: Generates natural language answers from query results
      - `QuestionDefinition.py`: Base class for question type definitions
      - `QuestionTypeSelector.py`: Selects appropriate question definition for user input
      - `SearchStrategy.py`: Defines search strategies for episodic memory queries
      - `Evaluator.py`: Evaluates verbalization quality
      - `PossibleClasses.py`: Registry of available question definition classes
      - `generate_init.py`: Utility for generating __init__.py files
      - `ManualQuestionDefinitions/`: Predefined question type implementations
        - `ActivitiesDuringTimeSpanQuestionDefinition.py`
        - `AffordancesCurrentSceneQuestionDefinition.py`
        - `LastActionPerformedQuestionDefinition.py`
        - `LastKnownLocationObjectQuestionDefinition.py`
        - `PreferencesOfPersonQuestionDefinition.py`
        - `load_classes.py`: Dynamic class loading for question definitions
      - `generation/`: Code generation utilities
        - `generate_llm_question_definition.py`: Generates LLM-based question definitions
        - `generate_manual_question_definition.py`: Generates manual question definition templates
        - `LLMQuestionDefinitionSkeleton.txt`: Template for LLM question definitions
        - `ManualQuestionDefinitionSkeleton.txt`: Template for manual question definitions
    - `util/`: Utility modules
      - `armarx_util.py`: ArmarX-specific utilities
      - `armem_util.py`: Episodic memory (ArmarX Memory) utilities
      - `FileUtil.py`: File I/O utilities
      - `semantic_util.py`: Semantic matching and processing
      - `python_function_generation_util.py`: Dynamic Python function generation
      - `generate_actions_objects_locations.py`: Generates action/object/location data
      - `generate_entities_type_data_prompt.py`: Generates entity type prompts
      - `generate_type_info_from_instance.py`: Extracts type information from instances
      - `llm_connector/`: LLM integration
        - `api_key_handling.py`: API key management
        - `llm_logger.py`: LLM interaction logging
        - `llm_verbalization_generation.py`: LLM-based verbalization generation
        - `match_checker.py`: Checks if questions match existing definitions
        - `prompt_construction.py`: Constructs LLM prompts
        - `query_llm.py`: LLM query interface
      - `parallelization_tools/`: Parallel processing utilities
        - `parallelization_util.py`: General parallelization utilities
        - `parallel_semantic_util.py`: Parallel semantic processing
      - `prompts/`: LLM prompt templates and data
        - Various `.txt` files: Prompt templates for different stages
        - `segment_descriptions.json`: Episodic memory segment descriptions
      - `util_data/`: Static data files
        - `possible_actions.json`: Available robot actions
        - `spoken_location_name_to_id_location.json`: Location name mappings
        - `spoken_name_to_id_objects.json`: Object name mappings
    - `config/`: Configuration management
      - `config.xml`: Configuration file
      - `read_config.py`: Configuration reader
      - `write_config.py`: Configuration writer
    - `tests/`: Unit tests
      - `test_answer_generator.py`: Tests for AnswerGenerator
    - `log/`: Runtime logs
      - `cost_log.csv`: LLM API cost tracking
      - `evaluation_log.csv`: Evaluation results
      - `StatMessages.txt`: Statistical messages

### Additional Directories

- `data/verbalization/`: ArmarX variant info and package data
- `scenarios/GeneralVerbalization/`: ArmarX scenario configuration
  - `GeneralVerbalization.scx`: Scenario definition
  - `config/`: Scenario-specific configuration
- `etc/`: CMake configuration and Doxygen documentation setup
- `build/`: Build artifacts (CMake-generated, gitignored)
- `python/verbalization/.venv/`: Python virtual environment

### Configuration Files

- `python/verbalization/pyproject.toml`: Poetry project configuration with dependencies
- `python/verbalization/poetry.toml`: Poetry settings
- `CMakeLists.txt`: Root CMake configuration
- `python/CMakeLists.txt`, `python/core/CMakeLists.txt`, `python/verbalization/CMakeLists.txt`: Python module CMake integration

---
> Source: [Humanoids-2025/verbalization](https://github.com/Humanoids-2025/verbalization) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
