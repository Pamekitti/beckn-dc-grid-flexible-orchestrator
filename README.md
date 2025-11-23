# DEG Compute Orchestrator

> LLM-orchestrated data center optimization for grid flexibility via Beckn Protocol

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![DEG Hackathon 2025](https://img.shields.io/badge/DEG%20Hackathon-2025-blue)](https://github.com/yourusername/deg-compute-orchestrator)

**Team ThermoTrace** | DEG Hackathon 2025 - Problem 2: Compute–Energy Convergence in a DEG World

---

## Overview

This project enables data centers to participate in grid flexibility markets by adding an **LLM orchestration layer** on top of multi-agent reinforcement learning control systems. We coordinate workload deferral, battery storage, and HVAC operations in response to carbon intensity forecasts and grid signals via **Beckn Protocol**.

### The Problem

Data centers consume 2.5% of UK electricity today, projected to reach 6% by 2030. Each large facility uses power equivalent to 100,000 households. As AI workloads grow, data centers need to:
- Minimize cost per inference under carbon intensity caps
- Participate in flexibility markets (UK P415)
- Coordinate complex subsystems (compute, cooling, storage)
- Provide auditable decisions for regulatory compliance

### Our Solution

We fork [HP Labs' SustainDC](https://github.com/HewlettPackard/dc-rl) benchmark and add:
1. **LLM Orchestrator** - Coordinates three RL agents with external grid signals
2. **Beckn Protocol Integration** - Enables grid communication and P415 participation
3. **Audit Trail System** - Natural language explanations for every decision

## Architecture

```mermaid
graph TB
    subgraph "External Grid (Beckn Protocol)"
        G1[Carbon Intensity<br/>Forecasts]
        G2[Energy Prices]
        G3[P415 Flexibility<br/>Requests]
    end

    subgraph "LLM Orchestration Layer"
        LLM[LLM Orchestrator<br/>Coordinates & Overrides]
    end

    subgraph "SustainDC RL Agents"
        A1[Load Shift Agent<br/>Workload Deferral]
        A2[Battery Agent<br/>Charge/Discharge]
        A3[HVAC Agent<br/>Cooling Control]
    end

    subgraph "Data Center"
        DC1[Workload Queue]
        DC2[Battery Storage]
        DC3[HVAC System]
    end

    subgraph "Outputs"
        O1[Coordinated<br/>Actions]
        O2[Audit Logs]
        O3[Beckn<br/>Confirmation]
    end

    G1 --> LLM
    G2 --> LLM
    G3 --> LLM
    
    A1 --> LLM
    A2 --> LLM
    A3 --> LLM
    
    DC1 -.-> A1
    DC2 -.-> A2
    DC3 -.-> A3
    
    LLM --> O1
    LLM --> O2
    LLM --> O3
    
    O1 --> DC1
    O1 --> DC2
    O1 --> DC3

    style LLM fill:#ff9999
    style A1 fill:#99ccff
    style A2 fill:#99ccff
    style A3 fill:#99ccff
```

### Key Components

**SustainDC Foundation (HP Labs)**
- Three specialized RL agents optimizing different data center subsystems
- Load Shift Agent: Minimizes workload delay costs
- Battery Agent: Optimizes energy storage usage
- HVAC Agent: Balances cooling energy with temperature limits

**The Coordination Gap**
- Agents observe only local state (workload queue, battery SoC, temperature)
- Cannot see external grid signals or market opportunities
- Optimize locally without system-wide coordination

**Beckn Grid Interface**
- Receives carbon intensity forecasts (UK Carbon Intensity API)
- Gets energy prices and P415 flexibility requests
- Publishes available deferrable capacity via Beckn catalog endpoints

**LLM Orchestrator**
- Reads both agent decisions AND external grid signals
- Detects conflicts (e.g., running workloads during high carbon periods)
- Identifies opportunities (P415 payments > deferral costs)
- Generates natural language audit trails for P415 settlement

## Impact

### Financial Benefits
- **Revenue**: P415 payments up to £3,000/MWh during grid stress
- **Cost Savings**: 15-35% reduction in energy bills through load shifting
- **Example**: Large data center (£36M annual energy cost) saves £5-12M/year

### Environmental Impact
- **Carbon Reduction**: Shifting 30% of workload from high-carbon (270 gCO2/kWh) to low-carbon windows (28 gCO2/kWh) cuts 21,780 tonnes CO2 annually
- **Equivalent**: Removing 4,700 cars from roads per data center

### Grid Benefits
- **Infrastructure Savings**: UK's 10-12 GW flexibility target by 2030 avoids £5-6 billion in peaker plant construction
- **Renewable Integration**: Absorbs solar/wind oversupply without curtailment

## Installation

### Prerequisites
```bash
Python 3.8+
Node.js 16+ (for Beckn integration)
CUDA 11.0+ (optional, for GPU acceleration)
```

### Setup

1. **Clone the repository**
```bash
git clone https://github.com/yourusername/deg-compute-orchestrator.git
cd deg-compute-orchestrator
```

2. **Install Python dependencies**
```bash
pip install -r requirements.txt
```

3. **Install SustainDC base environment**
```bash
cd sustaindc
pip install -e .
```

4. **Configure API keys**
```bash
cp .env.example .env
# Add your API keys:
# - UK Carbon Intensity API (free, no key needed)
# - LLM API key (OpenAI/Anthropic)
# - Beckn network credentials
```

5. **Run the orchestrator**
```bash
python orchestrator/main.py --config configs/default.yaml
```

## Usage

### Basic Example

```python
from orchestrator import LLMOrchestrator
from sustaindc import SustainDCEnv

# Initialize SustainDC environment
env = SustainDCEnv(
    location='london',
    workload_file='data/google_2011.csv'
)

# Initialize LLM orchestrator
orchestrator = LLMOrchestrator(
    llm_provider='anthropic',
    beckn_endpoint='https://beckn.network/gateway'
)

# Run coordination loop
for step in range(8760):  # One year, hourly
    # Get agent decisions
    agent_actions = env.get_agent_proposals()
    
    # Get grid signals via Beckn
    grid_signals = orchestrator.fetch_grid_signals()
    
    # Coordinate and execute
    coordinated_actions = orchestrator.coordinate(
        agent_actions, 
        grid_signals
    )
    
    # Execute in environment
    obs, reward, done, info = env.step(coordinated_actions)
    
    # Log audit trail
    orchestrator.log_decision(
        actions=coordinated_actions,
        reasoning=info['reasoning'],
        carbon_intensity=grid_signals['carbon']
    )
```

## Project Structure

```
deg-compute-orchestrator/
├── orchestrator/           # LLM coordination logic
│   ├── llm_coordinator.py
│   ├── beckn_client.py
│   └── audit_logger.py
├── sustaindc/             # Fork of HewlettPackard/dc-rl
│   ├── agents/
│   ├── envs/
│   └── data/
├── configs/               # Configuration files
├── examples/              # Usage examples
├── tests/                 # Unit tests
└── docs/                  # Documentation
```

## Beckn Protocol Integration

Our implementation uses Beckn Protocol for grid coordination:

**Discovery**: Data center publishes catalog of deferrable workloads
```json
{
  "descriptor": {
    "name": "Data Center Flexibility Catalog"
  },
  "items": [
    {
      "id": "ai-training-batch-1",
      "descriptor": {
        "name": "AI Training Workload Batch"
      },
      "quantity": {
        "available": { "count": 1000 }
      },
      "time": {
        "duration": "PT4H",
        "range": {
          "start": "2025-11-23T20:00:00Z",
          "end": "2025-11-24T08:00:00Z"
        }
      },
      "tags": [
        {
          "descriptor": { "name": "energy_consumption" },
          "list": [{ "value": "50MWh" }]
        },
        {
          "descriptor": { "name": "deferral_window" },
          "list": [{ "value": "12h" }]
        }
      ]
    }
  ]
}
```

**Select & Confirm**: Grid operator requests flexibility, data center confirms delivery

## Development

### Running Tests
```bash
pytest tests/
```

### Code Formatting
```bash
black orchestrator/
flake8 orchestrator/
```

### Contributing
This is a hackathon project, but we welcome improvements! Please open an issue first to discuss changes.

## Team

**ThermoTrace** - University College London
- Pamekitti Puktalae - AI Engineer
- Arhaan Shaikh - Fullstack Developer
- Sami Uwl - Fullstack Developer
- Vishal Sharma - Frontend Developer

## References

### Foundation
- **SustainDC**: HP Labs' Multi-Agent RL Benchmark for Sustainable Data Centers  
  GitHub: https://github.com/HewlettPackard/dc-rl

### Data Sources
- **UK Carbon Intensity API**: https://carbonintensity.org.uk
- **Beckn Protocol**: https://becknprotocol.io

### Regulatory Context
- **Ofgem P415**: Virtual Load Profile for Demand Response  
  https://www.ofgem.gov.uk/publications/direction-modify-balancing-and-settlement-code-p415-virtual-lead-parties

## License

This project is released under the **MIT License**.

### Our Contribution
All original work (LLM orchestrator, Beckn integration, audit logging) is MIT licensed.

### Foundation Work
Built on [SustainDC](https://github.com/HewlettPackard/dc-rl) by HP Labs (MIT License). We extend their three-agent system with coordination capabilities.

### Third-Party Components
- SustainDC benchmark suite (HP Labs, MIT License)
- Beckn Protocol specifications (Open standard)
- UK Carbon Intensity API (Public data)

---

**Built for DEG Hackathon 2025** | Problem 2: Compute–Energy Convergence in a DEG World
