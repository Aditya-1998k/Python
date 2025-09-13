**Problem with current Devops**:

1. `Linting and Formating` : Multiple tools (flake8, isort, black formater, pep8) which is slow and fragmented (+10 minutes cumulative)
2. `CI/CD`: YAML-heavy config (Jenkins, GitHub Actions) - Hard to reuse and locked in.
3. `Cloud Costs`: Inefficient pipelines (Money wasting)

**Solution** - `Ruff + Dagger`

### Ruff
1. Ruff is 100x faster than python linters and written purely in rust.
2. Replaces flake8, isort, pylint, pyflakes in one tool
3. Handles linting, formatting and imports together.

```bash
# Install Ruff
pip install ruff

# Check lint issues
ruff check .

# Auto-fix (imports, formatting and linting)
ruff check . --fix
```

**.pre-commit-config.yaml**
```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.6.5
    hooks:
      - id: ruff
        args: [--fix]
```

### Dagger
1. CI/CD as python code. No YAML lock-in.
2. Pipeline can run anywhere either in laptop, GitHub Actions, Circle CI, kubernetes
3. Support parallel execution (Lint, test, build at the same time)
4. With python function, we can reuse it.

```
import dagger
import anyio

async def main():
    async with dagger.Connection() as client:
        src = client.host().directory(".")

        # Run Ruff linting
        lint = await (
            client.container()
            .from_("python:3.11")
            .with_exec(["pip", "install", "ruff"])
            .with_mounted_directory("/src", src)
            .with_workdir("/src")
            .with_exec(["ruff", "check", "."])
            .stdout()
        )
        print("✅ Ruff Lint Result:\n", lint)

anyio.run(main)
```
1. Spin up a python:3.11 container.
2. Install Ruff.
3. Mount your project directory inside /src.
4. Run ruff check . on your code.

Explaination of step 1:
```
Container → a lightweight, isolated environment (like a mini virtual machine, but faster)
created using a Docker image.
python:3.11 → the base image from Docker Hub that comes with Python 3.11 pre-installed.
Spin up → Dagger pulls that image and runs it temporarily, just for your pipeline step.
```
