# Contributing to PySensemakr

We welcome contributions to `PySensemakr`. This document provides guidelines for contributing to the project.

## Getting Started

1. Fork the repository on GitHub.
2. Clone your fork locally:
   ```bash
   git clone https://github.com/<your-username>/PySensemakr.git
   cd PySensemakr
   ```
3. Install the package in development mode:
   ```bash
   pip install -e .
   ```

## Development Workflow

1. Create a new branch for your feature or bug fix:
   ```bash
   git checkout -b feature/your-feature-name
   ```
2. Make your changes and add tests as appropriate.
3. Run the test suite to ensure all tests pass:
   ```bash
   pytest
   ```
4. Commit your changes with a clear commit message.
5. Push to your fork and submit a pull request.

## Reporting Issues

If you encounter a bug or have a feature request, please open an issue on the GitHub issue tracker. When reporting bugs, please include:

- A minimal reproducible example.
- The output you expected and the output you received.
- Your Python version and operating system.

## Code Style

- Follow PEP 8 conventions.
- Include docstrings for public functions and classes.
- Add unit tests for new functionality.

## Testing

We use `pytest` for testing. To run the full test suite:

```bash
pytest
```

Code coverage is monitored via Codecov. Please ensure that new code is covered by tests.

## License

By contributing to `PySensemakr`, you agree that your contributions will be licensed under the MIT License.
