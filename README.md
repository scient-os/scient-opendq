<div align="center">

![Scient Logo](scient.svg)

# OpenDQ - Open Source Data Quality Framework

### Coming Soon

---

**OpenDQ** is an intelligent, AI-powered data quality validation framework that automatically maps your data to ontology-based quality rules and validates them using embedded Python code execution.

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-coming%20soon-orange.svg)]()

</div>

---

## What is OpenDQ?

OpenDQ brings intelligent automation to data quality validation by combining:

- **AI-Powered Field Mapping** - Uses LLMs and feature vectors to intelligently map columns to ontology properties
- **Statistical Profiling** - Automatically profiles your data to understand patterns and distributions
- **Ontology-Based Validation** - Supports FHIR, Schema.org, and custom ontologies
- **Embedded Python Rules** - Execute flexible validation logic with Python code in rules
- **Rich Reporting** - Generate detailed HTML reports with pass/fail statistics
- **Multiple Data Sources** - File (CSV, Parquet), databases, S3, and custom connectors

## Key Features

### AI-Powered Field Mapping
Automatically maps your data columns to ontology properties using:
- LLM-powered semantic understanding
- Feature vector analysis from data profiling
- Support for FHIR, Schema.org, and custom ontologies

### Statistical Profiling
Automatically analyzes your data to extract:
- Data types and distributions
- Pattern recognition (emails, phones, dates)
- Statistical metrics (min, max, mean, std dev)
- Top values and frequencies

### Flexible Validation Rules
Execute validation logic using embedded Python code:
- Format validation (dates, emails, phone numbers)
- Range checks and constraints
- Custom business rules
- Cross-field validation

### Rich Reporting
Generate comprehensive HTML reports with:
- Pass/fail statistics by record, rule, and field
- Detailed error messages for failed records
- Visual summaries and charts
- Export results programmatically

## Quick Start

### Simple One-Line API

```python
from scient.opendq.connectors import create_connection
from scient.opendq.executor import RuleExecutorFactory
from scient.opendq.reports import generate_dq_report

# 1. Create a connector to your data
with create_connection("file", file_path="data/customers.csv") as connector:

    # 2. Create executor with intelligent defaults
    # - Auto-generates feature vectors from data
    # - Uses LLM-powered field mapping
    # - Loads FHIR ontology rules by default
    executor = RuleExecutorFactory.create_default(connector)

    # 3. Run data quality checks
    result = executor.run()

    # 4. Generate HTML report
    generate_dq_report(result, output_path="reports/dq_report.html")

    # 5. Access results programmatically
    print(f"Total Records: {result['summary']['records']['total_count']}")
    print(f"Failed Records: {result['summary']['records']['failed_count']}")
    print(f"Pass Rate: {result['summary']['records']['pass_percentage']}%")
```

### Different Data Sources

```python
# CSV file
with create_connection("file", file_path="data.csv") as connector:
    executor = RuleExecutorFactory.create_default(connector)
    result = executor.run()

# Parquet file
with create_connection("file", file_path="data.parquet") as connector:
    executor = RuleExecutorFactory.create_default(connector)
    result = executor.run()

# Database (Coming Soon)
with create_connection("postgresql", host="localhost", database="mydb") as connector:
    executor = RuleExecutorFactory.create_default(connector)
    result = executor.run()
```

### Custom Field Mapping

```python
from scient.opendq.field_mapper import DictFieldMapper
from scient.opendq.executor import RuleExecutionContext, RuleExecutor

# Define explicit mappings
mappings = {
    "patient_id": "http://hl7.org/fhir/Patient.id",
    "first_name": "http://hl7.org/fhir/Patient.name.given",
    "last_name": "http://hl7.org/fhir/Patient.name.family",
    "birth_date": "http://hl7.org/fhir/Patient.birthDate"
}

mapper = DictFieldMapper(field_mappings=mappings)
context = RuleExecutionContext(field_mapper=mapper)
executor = RuleExecutor(connector, context)
result = executor.run()
```

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    RuleExecutorFactory                      │
│  Orchestrates: Connectors → Profiling → Mapping → Rules     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
        ┌──────────────────────────────────────┐
        │        RuleExecutor                  │
        │  Executes validation rules on data   │
        └──────────────────────────────────────┘
                              │
        ┌─────────────────────┴──────────────────────┐
        │                     │                      │
        ▼                     ▼                      ▼
┌──────────────┐    ┌─────────────────┐    ┌──────────────┐
│  Connector   │    │  FieldMapper    │    │  RuleStore   │
│              │    │                 │    │              │
│ - File       │    │ - FVFieldMapper │    │ - FHIR       │
│ - Database   │    │ - AutoMapper    │    │ - Schema.org │
│ - S3         │    │ - DictMapper    │    │ - Custom     │
└──────────────┘    └─────────────────┘    └──────────────┘
```

### How It Works

1. **Data Profiling**: Factory profiles your data to extract statistical features (data types, distributions, patterns)
2. **Field Mapping**: AI analyzes feature vectors + ontology schemas to map columns to properties
3. **Rule Loading**: Loads validation rules from ontology-specific rule stores
4. **Validation**: Executes Python validation functions against your data
5. **Reporting**: Generates detailed reports with pass/fail statistics

## Field Mapper Comparison

| Mapper | Use Case | Requires LLM | Accuracy | Setup Time |
|--------|----------|--------------|----------|------------|
| **FVFieldMapper** | Production use, automatic mapping | ✅ Yes | 🟢 High | ⚡ Auto |
| **AutoFieldMapper** | Quick tests, fuzzy matching | ❌ No | 🟡 Medium | ⚡ Instant |
| **DictFieldMapper** | Known schemas, manual control | ❌ No | 🟢 Perfect | 🐌 Manual |

**Recommendation**: Use `RuleExecutorFactory.create_default()` which uses **FVFieldMapper** with auto-generated feature vectors for the best balance of accuracy and ease of use.

## Understanding Results

The `executor.run()` method returns a comprehensive dictionary with validation results:

```json
{
  "summary": {
    "records": {
      "total_count": 1000,
      "failed_count": 45,
      "passed_count": 955,
      "failed_percentage": 4.5,
      "failed_indices": [12, 34, 56, ...]
    },
    "rules": {
      "total_count": 25,
      "failed_count": 8,
      "passed_count": 17,
      "failed_percentage": 32.0
    },
    "fields": {
      "patient_id": {
        "mapped_property_uri": "http://hl7.org/fhir/Patient/identifier",
        "total_count": 1000,
        "passed_count": 1000,
        "failed_count": 0
      },
      "birth_date": {
        "mapped_property_uri": "http://hl7.org/fhir/Patient/birthDate",
        "total_count": 1000,
        "passed_count": 955,
        "failed_count": 45
      }
    }
  },
  "error_records": [
    {
      "index": 12,
      "record": {"birth_date": "02/31/1985", ...},
      "errors": [...]
    }
  ]
}
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `SCIENT_OSS_HOME` | Root directory for OpenDQ resources | Project root |
| `OPENAI_API_KEY` | OpenAI API key for GPT models | Required for FVFieldMapper with OpenAI |
| `ANTHROPIC_API_KEY` | Anthropic API key for Claude models | Required for FVFieldMapper with Claude |

LiteLLM automatically detects provider-specific API keys for all major LLM providers.

## What's Coming

This repository is currently being prepared for public release. We're working on:

- Full source code release with installation instructions
- Complete documentation and API reference
- Sample datasets and step-by-step tutorials
- Extended connector support (PostgreSQL, MySQL, S3, Azure)
- Additional ontology integrations
- Enhanced reporting capabilities with visualizations
- Community contribution guidelines
- Example projects and use cases

## Installation (Preview)

Once released, installation will be simple using [uv](https://github.com/astral-sh/uv):

```bash
# Clone the repository
git clone https://github.com/scient-os/scient-opendq.git
cd scient-opendq

# Install dependencies with uv
uv sync

# Run example
export SCIENT_OSS_HOME=/path/to/scient-opendq
export OPENAI_API_KEY="sk-..."
PYTHONPATH=src python main.py
```

## Use Cases

- **Healthcare Data Validation**: Validate patient records against FHIR standards
- **Data Migration**: Ensure data quality before/after migrations
- **Data Warehousing**: Validate ETL pipeline outputs
- **API Data Validation**: Check incoming data against schema definitions
- **Compliance Checking**: Ensure data meets regulatory requirements
- **Data Quality Monitoring**: Continuous validation in production pipelines

## Stay Updated

This project will be available soon at:

**GitHub**: [github.com/scient-oss/scient-opendq](https://github.com/scient-oss/scient-opendq)

Watch this repository to get notified when we launch.

## Related Projects

- **scient-commons** - Common utilities and abstractions (Coming Soon)
- **scient-connectors** - Data connector implementations (Coming Soon)

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Uses [LiteLLM](https://github.com/BerriAI/litellm) for unified LLM access
- Uses [uv](https://github.com/astral-sh/uv) for fast dependency management
- Built on top of open ontology standards (FHIR, Schema.org)

---

<div align="center">

**Built with ❤️ by the Scient team**

[Enterprise AI](https://scient.com)

</div>
