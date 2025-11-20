# ai-models

<p align="center">
  <a href="https://github.com/ecmwf/codex/raw/refs/heads/main/Project%20Maturity">
    <img src="https://github.com/ecmwf/codex/raw/refs/heads/main/Project%20Maturity/sandbox_badge.svg" alt="Static Badge">
  </a>

<a href="https://codecov.io/gh/ecmwf/earthkit-data">
    <img src="https://codecov.io/gh/ecmwf-lab/ai-models/branch/main/graph/badge.svg" alt="Code Coverage">
  </a>

<a href="https://opensource.org/licenses/apache-2-0">
    <img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg" alt="License: Apache 2.0">
  </a>

<a href="https://github.com/ecmwf/earthkit-data/releases">
    <img src="https://img.shields.io/github/v/release/ecmwf-lab/ai-models?color=blue&label=Release&style=flat-square" alt="Latest Release">
  </a>
</p>

**DISCLAIMER**

> \[!IMPORTANT\]
> This project is **BETA** and will be **Experimental** for the foreseeable future.
> Interfaces and functionality are likely to change, and the project itself may be scrapped.
> **DO NOT** use this software in any project/software that is operational.

## Overview

The `ai-models` package is a Python framework for running AI-based weather forecasting models. It provides a unified command-line interface and plugin architecture for executing various AI weather prediction models with flexible input sources and output formats.

## Key Features

- **Modular Architecture**: Plugin-based system supporting multiple AI weather models
- **Multiple Input Sources**: Fetch data from MARS, CDS, ECMWF Open Data, or local GRIB files
- **Flexible Output**: Write forecasts to GRIB files with customizable metadata
- **Remote Execution**: Support for running models on remote inference servers
- **Asset Management**: Automated downloading and management of model weights and assets
- **GPU Acceleration**: Optimized for GPU execution with fallback to CPU

## Usage

Although the source code `ai-models` and its plugins are available under open sources licences, some model weights may be available under a different licence. For example some models make their weights available under the CC-BY-NC-SA 4.0 license, which does not allow commercial use. For more informations, please check the license associated with each model on their main home page, that we link from each of the corresponding plugins.

## Project Structure

```
ai-models/
├── src/
│   └── ai_models/
│       ├── __init__.py           # Package initialization
│       ├── __main__.py           # CLI entry point and argument parsing
│       ├── model.py              # Base Model class and core functionality
│       ├── stepper.py            # Forecast stepping and timing utilities
│       ├── checkpoint.py         # Model checkpoint inspection utilities
│       ├── inputs/               # Input data source implementations
│       │   ├── __init__.py
│       │   ├── base.py          # Base class for request-based inputs
│       │   ├── file.py          # Local GRIB file input
│       │   ├── mars.py          # ECMWF MARS archive input
│       │   ├── cds.py           # Copernicus CDS input
│       │   ├── opendata.py      # ECMWF Open Data input
│       │   ├── compute.py       # Field computation utilities
│       │   ├── interpolate.py   # Spatial interpolation
│       │   ├── recenter.py      # Data recentering utilities
│       │   └── transform.py     # Data transformation utilities
│       ├── outputs/              # Output handling
│       │   └── __init__.py      # GRIB output writers
│       └── remote/               # Remote execution support
│           ├── __init__.py
│           ├── api.py           # Remote API client
│           ├── config.py        # Remote configuration management
│           └── model.py         # Remote model wrapper
├── tests/                        # Test suite
│   ├── requirements.txt
│   └── test_code.py
├── pyproject.toml               # Project metadata and dependencies
└── README.md                    # This file
```

## Core Components

### Model System
- **`model.py`**: Defines the base `Model` class with methods for running forecasts, managing assets, handling input/output, and tracking provenance
- **`stepper.py`**: Provides the `Stepper` class for iterating through forecast steps with progress tracking and ETA calculation
- **`checkpoint.py`**: Utilities for inspecting PyTorch checkpoint files without loading full tensors

### Input Sources
The framework supports multiple data sources via a plugin architecture:
- **MARS**: ECMWF's meteorological archive (default)
- **CDS**: Copernicus Climate Data Store (ERA5 reanalysis)
- **OpenData**: ECMWF's open data service
- **File**: Local GRIB files

All input sources implement field retrieval for surface (`sfc`), pressure level (`pl`), and model level (`ml`) data.

### Output System
- **GRIB Output**: Writes forecast fields to GRIB format with configurable metadata
- **Split Output**: Option to write separate files per time step or parameter
- **Metadata Control**: Custom GRIB keys for experiment version, class, stream, etc.

### Remote Execution
- **`remote/api.py`**: Client for submitting inference jobs to remote servers
- **`remote/model.py`**: Wrapper that adapts local model interface to remote execution
- **Authentication**: Token-based authentication via configuration file or environment variables

## Prerequisites

Before using the `ai-models` command, ensure you have the following prerequisites:

- Python 3.9 or later (tested primarily with Python 3.10 on Linux/MacOS)
- An ECMWF and/or CDS account for accessing input data (see below for more details)
- A computer with a GPU for optimal performance (strongly recommended)

## Installation

To install the `ai-models` command, run the following command:

```bash
pip install ai-models
```

## Available Models

Currently, four models can be installed:

```bash
pip install ai-models-panguweather
pip install ai-models-fourcastnet
pip install ai-models-graphcast  # Install details at https://github.com/ecmwf-lab/ai-models-graphcast
pip install ai-models-fourcastnetv2
```

See [ai-models-panguweather](https://github.com/ecmwf-lab/ai-models-panguweather), [ai-models-fourcastnet](https://github.com/ecmwf-lab/ai-models-fourcastnet),
 [ai-models-fourcastnetv2](https://github.com/ecmwf-lab/ai-models-fourcastnetv2) and [ai-models-graphcast](https://github.com/ecmwf-lab/ai-models-graphcast) for more details about these models.

## Running the models

To run model, make sure it has been installed, then simply run:

```bash
ai-models <model-name>
```

Replace `<model-name>` with the name of the specific AI model you want to run.

By default, the model will be run for a 10-day lead time (240 hours), using yesterday's 12Z analysis from ECMWF's MARS archive.

To produce a 15 days forecast, use the `--lead-time HOURS` option:

```bash
ai-models --lead-time 360 <model-name>
```

You can change the other defaults using the available command line options, as described below.

## Performances Considerations

The AI models can run on a CPU; however, they perform significantly better on a GPU. A 10-day forecast can take several hours on a CPU but only around one minute on a modern GPU.

:warning: **We strongly recommend running these models on a computer equipped with a GPU for optimal performance.**

It you see the following message when running a model, it means that the ONNX runtime was not able to find a the CUDA libraries on your system:
> [W:onnxruntime:Default, onnxruntime_pybind_state.cc:541 CreateExecutionProviderInstance] Failed to create CUDAExecutionProvider. Please reference <https://onnxruntime.ai/docs/reference/execution-providers/CUDA-ExecutionProvider.html#requirements> to ensure all dependencies are met.

To fix this issue, we suggest that you install `ai-models` in a [conda](https://docs.conda.io/en/latest/) environment and install the CUDA libraries in that environment. For example:

```bash
conda create -n ai-models python=3.10
conda activate ai-models
conda install cudatoolkit
pip install ai-models
...
```

## Assets

The AI models rely on weights and other assets created during training. The first time you run a model, you will need to download the trained weights and any additional required assets.

To download the assets before running a model, use the following command:

```bash
ai-models --download-assets <model-name>
```

The assets will be downloaded if needed and stored in the current directory. You can provide a different directory to store the assets:

```bash
ai-models --download-assets --assets <some-directory> <model-name>
```

Then, later on, simply use:

```bash
ai-models --assets <some-directory>  <model-name>
```

or

```bash
export AI_MODELS_ASSETS=<some-directory>
ai-models <model-name>
```

For better organisation of the assets directory, you can use the `--assets-sub-directory` option. This option will store the assets of each model in its own subdirectory within the specified assets directory.

## Input data

The models require input data (initial conditions) to run. You can provide the input data using different sources, as described below:

### From MARS

By default, `ai-models`  use yesterday's 12Z analysis from ECMWF, fetched from the Centre's MARS archive using the [ECMWF WebAPI](https://www.ecmwf.int/en/computing/software/ecmwf-web-api). You will need an ECMWF account to access that service.

To change the date or time, use the `--date` and `--time` options, respectively:

```bash
ai-models --date YYYYMMDD --time HHMM <model-name>
```

### From the CDS

You can start the models using ERA5 (ECMWF Reanalysis version 5) data for the [Copernicus Climate Data Store (CDS)](https://cds.climate.copernicus.eu/). You will need to create an account on the CDS. The data will be downloaded using the [CDS API](https://cds.climate.copernicus.eu/api-how-to).

To access the CDS, simply add `--input cds` on the command line. Please note that ERA5 data is added to the CDS with a delay, so you will also have to provide a date with `--date YYYYMMDD`.

```bash
ai-models --input cds --date 20230110 --time 0000 <model-name>
```

### From a GRIB file

If you have input data in the GRIB format, you can provide the file using the `--file` option:

```bash
ai-models --file <some-grib-file> <model-name>
```

The GRIB file can contain more fields than the ones required by the model. The `ai-models` command will automatically select the necessary fields from the file.

To find out the list of fields needed by a specific model as initial conditions, use the following command:

```bash
 ai-models --fields <model-name>
 ```

## Output

By default, the model output will be written in GRIB format in a file called `<model-name>.grib`. You can change the file name with the option `--path <file-name>`. If the path you specify contains placeholders between `{` and `}`, multiple files will be created based on the [eccodes](https://confluence.ecmwf.int/display/ECC) keys. For example:

```bash
 ai-models --path 'out-{step}.grib' <model-name>
 ```

This command will create a file for each forecasted time step.

If you want to disable writing the output to a file, use the `--output none` option.

## Command Line Options

### General Options

- `--help`: Display help message
- `--models`: List all installed models
- `--version`: Print ai-models version and exit
- `--debug`: Enable debug mode for verbose logging
- `-v, --verbose`: Increase verbosity (can be repeated)

### Input Options

- `--input INPUT`: Input source (`mars`, `cds`, `ecmwf-open-data`, `opendata`, or `file`)
- `--file FILE`: Path to input GRIB file (automatically sets `--input file`)
- `--date DATE`: Analysis date for the forecast (default: `-1` for yesterday)
- `--time TIME`: Analysis time in hours (default: `12`)

### Output Options

- `--output OUTPUT`: Output destination (`file` or `none`)
- `--path PATH`: Path for model output (default: `<model-name>.grib`)
- `--expver EXPVER`: Experiment version metadata
- `--class CLASS`: GRIB 'class' metadata
- `--metadata KEY=VALUE`: Additional GRIB metadata (can be repeated)

### Forecast Options

- `--lead-time HOURS`: Forecast length in hours (default: `240` for 10 days)
- `--hindcast-reference-year YEAR`: Reference year for hindcast encoding
- `--hindcast-reference-date DATE`: Reference date for hindcast encoding
- `--staging-dates DATES`: Staging dates for hindcast encoding

### Asset Management

- `--assets ASSETS`: Path to directory containing model weights and assets (default: current directory or `$AI_MODELS_ASSETS`)
- `--assets-sub-directory / --no-assets-sub-directory`: Organize assets in `<assets>/<model-name>` subdirectories
- `--assets-list`: List assets required by the model
- `--download-assets`: Download missing assets before running

### Model Information

- `--fields`: Display input fields required by the model
- `--retrieve-requests`: Print MARS retrieve requests to stdout
- `--archive-requests FILE`: Save MARS archive requests to file
- `--requests-extra KEY=VALUE,KEY=VALUE`: Extend or override retrieve/archive requests
- `--json`: Output requests in JSON format
- `--retrieve-fields-type TYPE`: Type of fields to retrieve (`constants`, `prognostics`, or `all`)
- `--retrieve-only-one-date`: Only retrieve the last date/time
- `--dump-provenance FILE`: Save provenance information to file

### Execution Options

- `--num-threads N`: Number of threads (model-specific, default: `1`)
- `--only-gpu`: Fail if GPU is not available
- `--deterministic`: Enable deterministic mode
- `--remote`: Enable remote execution (uses `~/.config/ai-models/api.yaml` or `$AI_MODELS_REMOTE=1`)

### Advanced Options

- `--model-version VERSION`: Model version to use (default: `latest`)

## Entry Points

The package defines several entry points for extensibility:

### Input Plugins (`ai_models.input`)
- `file`: FileInput
- `mars`: MarsInput
- `cds`: CdsInput
- `ecmwf-open-data`: OpenDataInput
- `opendata`: OpenDataInput

### Output Plugins (`ai_models.output`)
- `file`: FileOutput
- `none`: NoneOutput

### CLI Command
- `ai-models`: Main command-line interface

## Dependencies

The project relies on the following key dependencies (see `pyproject.toml` for complete list):

- **earthkit-data** (≥0.11.3): Data handling and GRIB I/O
- **earthkit-meteo**: Meteorological computations
- **earthkit-regrid**: Spatial interpolation and regridding
- **eccodes** (≥2.37): GRIB encoding/decoding library
- **numpy** (<2): Numerical operations
- **cdsapi**: CDS API client
- **ecmwf-api-client**: ECMWF Web API client
- **ecmwf-opendata**: ECMWF Open Data client
- **entrypoints**: Plugin discovery
- **gputil**: GPU utilities
- **multiurl**: Multi-source downloads
- **pyyaml**: YAML configuration
- **requests**: HTTP client
- **tqdm**: Progress bars

## Development

### Running Tests

```bash
cd tests
pip install -r requirements.txt
python test_code.py
```

### Creating a Plugin Model

To create your own AI weather model plugin:

1. Create a new package (e.g., `ai-models-mymodel`)
2. Define a model class inheriting from `ai_models.model.Model`
3. Implement required methods:
   - `__init__()`: Initialize model and load weights
   - `run()`: Execute the forecast
   - Set class attributes: `param_level_ml`, `param_level_pl`, `param_sfc`
4. Register as an entry point in your `pyproject.toml`:
   ```toml
   [project.entry-points."ai_models.model"]
   mymodel = "ai_models_mymodel:MyModel"
   ```

See existing model plugins for examples:
- [ai-models-panguweather](https://github.com/ecmwf-lab/ai-models-panguweather)
- [ai-models-fourcastnet](https://github.com/ecmwf-lab/ai-models-fourcastnet)
- [ai-models-graphcast](https://github.com/ecmwf-lab/ai-models-graphcast)
- [ai-models-fourcastnetv2](https://github.com/ecmwf-lab/ai-models-fourcastnetv2)

## Remote Execution

The framework supports running models on remote inference servers:

### Configuration

Create `~/.config/ai-models/api.yaml`:
```yaml
url: https://your-server.example.com/
token: your-auth-token
```

Or use environment variables:
```bash
export AI_MODELS_REMOTE_URL=https://your-server.example.com/
export AI_MODELS_REMOTE_TOKEN=your-auth-token
export AI_MODELS_REMOTE=1
```

### Usage

```bash
ai-models --remote <model-name>
ai-models --models --remote  # List available remote models
```

The remote execution workflow:
1. Uploads input GRIB file to server
2. Submits inference job with configuration
3. Polls for completion
4. Downloads and processes results

## License

```
Copyright 2022, European Centre for Medium Range Weather Forecasts.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

In applying this licence, ECMWF does not waive the privileges and immunities
granted to it by virtue of its status as an intergovernmental organisation
nor does it submit to any jurisdiction.
```
