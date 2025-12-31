# Run Python Unit Tests

A reusable GitHub Actions action for running Python unit tests with pytest, coverage reporting, and test result publishing.

## Features

- Runs pytest with code coverage
- Enforces minimum coverage threshold (configurable, default 80%)
- Uploads coverage reports to Codecov (optional token)
- Uploads coverage artifacts (XML, HTML, JUnit)
- Publishes test results as GitHub Check Runs
- Supports pip caching for faster builds
- Configurable Python version
- Flexible module path and requirements file location

## Usage

### Basic Usage

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Python Tests
        uses: ./.github/actions/run-python-tests
        with:
          module_path: 'api'
```

### With Custom Python Version

```yaml
- name: Run Python Tests
  uses: ./.github/actions/run-python-tests
  with:
    module_path: 'api'
    python_version: '3.12'
```

### With Codecov Token

```yaml
- name: Run Python Tests
  uses: ./.github/actions/run-python-tests
  with:
    module_path: 'api'
    codecov_token: ${{ secrets.CODECOV_TOKEN }}
    codecov_flags: 'api'
    codecov_name: 'api-tests'
```

### With Custom Coverage Threshold

```yaml
- name: Run Python Tests
  uses: ./.github/actions/run-python-tests
  with:
    module_path: 'api'
    coverage_threshold: '90'
```

### With Custom Requirements File

```yaml
- name: Run Python Tests
  uses: ./.github/actions/run-python-tests
  with:
    module_path: 'src'
    requirements_file: 'requirements-dev.txt'
```

### With Additional Pytest Arguments

```yaml
- name: Run Python Tests
  uses: ./.github/actions/run-python-tests
  with:
    module_path: 'api'
    pytest_args: '-v --tb=short'
```

### Matrix Strategy Example

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        module: [api, cronjob]
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Python Tests
        uses: ./.github/actions/run-python-tests
        with:
          module_path: ${{ matrix.module }}
          check_name: Unit Test Results (${{ matrix.module }})
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `module_path` | Path to the Python module directory | Yes | - |
| `python_version` | Python version to use | No | `3.13` |
| `coverage_threshold` | Minimum code coverage percentage | No | `80` |
| `codecov_token` | Codecov token (optional) | No | - |
| `codecov_flags` | Codecov flags for this module | No | - |
| `codecov_name` | Codecov name/identifier | No | `module_path` |
| `check_name` | Name for the GitHub check run | No | `Unit Test Results` |
| `requirements_file` | Path to requirements.txt relative to module_path | No | `requirements.txt` |
| `pytest_args` | Additional arguments to pass to pytest | No | `''` |

## Outputs

| Output | Description |
|--------|-------------|
| `coverage_percentage` | Code coverage percentage achieved |
| `test_result` | Test result status (`success` or `failure`) |

## Requirements

Your Python module should have:

1. **`requirements.txt`** (or custom path) with testing dependencies:
   ```txt
   pytest
   pytest-cov
   pytest-asyncio  # if using async tests
   pytest-mock     # if using mocks
   ```

2. **`pytest.ini`** or **`pyproject.toml`** with pytest configuration:
   ```ini
   [tool.pytest.ini_options]
   testpaths = ["tests"]
   pythonpath = ["."]
   addopts = "--cov=app --cov-report=xml --cov-report=html --cov-report=term --junit-xml=junit.xml --cov-fail-under=80"
   ```

3. **`.coveragerc`** (optional) for coverage configuration:
   ```ini
   [run]
   source = app
   omit = 
       tests/*
       */__pycache__/*
   ```

## Notes

- The action will fail if coverage is below the threshold (configured in `pytest.ini`)
- Coverage reports are always uploaded as artifacts, even if tests fail
- Codecov upload is optional and will be skipped if no token is provided
- Test results are published as GitHub Check Runs for easy viewing in PRs
- Pip caching is automatically enabled for faster builds

