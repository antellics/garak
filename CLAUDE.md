# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

garak is a LLM vulnerability scanner that checks if language models can be made to fail in undesirable ways. It probes for hallucination, data leakage, prompt injection, misinformation, toxicity generation, jailbreaks, and other weaknesses. Think of it as similar to nmap or Metasploit Framework, but for LLMs.

## Key Commands

### Running garak
```bash
# Run with default probes on a model
python -m garak --model_type <type> --model_name <name>

# Run specific probes
python -m garak --model_type <type> --model_name <name> --probes <probe_name>

# List available components
python -m garak --list_probes
python -m garak --list_detectors
python -m garak --list_generators
```

### Testing
```bash
# Install test dependencies first
pip install -e ".[tests]"

# Run all tests
python -m pytest

# Run specific test file
python -m pytest tests/probes/test_probes.py

# Run tests with coverage
python -m pytest --cov=garak

# Test a new probe with blank generator
python -m garak -m test.Blank -p mymodule -d always.Pass

# Test a new detector with blank probe  
python -m garak -m test.Blank -p test.Blank -d mymodule
```

### Code Quality
```bash
# Install lint dependencies
pip install -e ".[lint]"

# Format code with black
black garak/

# Run linter
pylint garak/
```

## Architecture Overview

garak follows a plugin-based architecture where components are dynamically loaded and interact through well-defined interfaces:

### Core Flow
1. **CLI Entry** (`garak/cli.py`) - Parses arguments and orchestrates the run
2. **Harness** (`garak/harnesses/`) - Coordinates the testing flow between components
3. **Generator** (`garak/generators/`) - Interfaces with the target LLM being tested
4. **Probe** (`garak/probes/`) - Generates test prompts to elicit specific behaviors
5. **Detector** (`garak/detectors/`) - Analyzes outputs to identify vulnerabilities
6. **Evaluator** (`garak/evaluators/`) - Aggregates results and produces reports

### Plugin System
- All plugins inherit from base classes in their respective `base.py` files
- Plugins are discovered dynamically via `_plugins.py`
- Each probe declares its `recommended_detectors` for automatic pairing
- Configuration is handled through YAML files and command-line overrides

### Key Design Patterns
- **Attempt Pattern**: Each probe generates `Attempt` objects containing prompts and tracking their lifecycle through generation and detection
- **Configurable Mixin**: Plugins use `garak.configurable.Configurable` for standardized configuration handling
- **Resource Loading**: Large datasets and models are loaded from `garak/data/` and `garak/resources/`

### State Management
- Run configuration stored in `_config` module with transient and persistent sections
- Results logged to JSONL files for analysis
- Parallel execution supported via `parallel_requests` and `parallel_attempts` settings

## Plugin Development

When creating new plugins:
1. Inherit from the appropriate base class (e.g., `garak.probes.base.TextProbe`)
2. Override minimal methods - leverage base class functionality
3. Define `recommended_detectors` for probes
4. Place resources in `garak/data/` or load from Hugging Face Hub
5. Follow existing naming conventions and code style

## Important Files
- `garak/_config.py` - Configuration management
- `garak/_plugins.py` - Plugin discovery and loading
- `garak/attempt.py` - Core Attempt data structure
- `garak/harnesses/probewise.py` - Default testing harness
- `garak/report.py` - Report generation