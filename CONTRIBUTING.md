# Contributing to Project Polaris

## Local Development Setup

### Prerequisites

- CMake 3.20 or higher
- C++20 compatible compiler (GCC 10+, Clang 12+, or MSVC 2019+)
- Git

### Building Locally

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd project-polaris
   ```

2. Configure and build:
   ```bash
   cmake -S . -B build
   cmake --build build
   ```

3. Run tests (when available):
   ```bash
   cd build
   ctest
   ```

4. Run the hello application:
   ```bash
   ./build/apps/hello/hello
   ```

### Development Workflow

1. Create a feature branch from `main`
2. Make your changes
3. Format code with clang-format
4. Run tests to ensure nothing is broken
5. Submit a pull request

### Code Style

- Follow the project's `.clang-format` configuration
- Use meaningful variable and function names
- Add comments for complex logic
- Follow C++20 best practices

### Commit Messages

Use conventional commit format:
- `feat(component): description` for new features
- `fix(component): description` for bug fixes
- `perf(component): description` for performance improvements