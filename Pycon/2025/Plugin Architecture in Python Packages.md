## Understanding Plugin Architecture for Python Packages

#### **What and Why Plugin Architecture?**
Plugin = Extension mechanism: A way to add new functionality without modifying the core package.

#### Why plugins?
1. Keeps core codebase small and maintainable.
2. Allows 3rd parties to extend functionality.
3. Widely used in tools like SQLAlchemy (dialects), Airflow (providers), Flask (extensions), pytest (plugins).

**Analogy**: Think of plugins like “USB ports” in your software. You define the port, anyone can build the device.

---

#### **SQLAlchemy Dialects as Example**
SQLAlchemy doesn’t hardcode Postgres/MySQL/SQLite. Instead, it defines a dialect interface.
A dialect translates SQLAlchemy Core expressions → database-specific SQL.

Example:
```python
from sqlalchemy import create_engine

engine = create_engine("postgresql://user:pass@localhost/db")
```
Here, `postgresql://` triggers SQLAlchemy to load the Postgres dialect plugin.
New dialects can be added via plugins (sqlalchemy-clickhouse, sqlalchemy-druid, etc).

This is how `SQLAlchemy` remains extensible to new databases without changing core.

**How SQLAlchemy Loads Plugins**
SQLAlchemy uses entry points `from importlib.metadata` to discover dialects.

Example (simplified from SQLAlchemy codebase):
```python
from importlib.metadata import entry_points

# Discover plugins registered under "sqlalchemy.dialects"
dialects = entry_points().select(group="sqlalchemy.dialects")

for ep in dialects:
    print(ep.name, ep.value)
    plugin_cls = ep.load()   # dynamically import the plugin

```
This lets SQLAlchemy say:

“Show me all dialects anyone has installed on this system.”

### Demo Example
```
plugin-demo/
├── hello-world/
│   ├── setup.py
│   └── hello_world/
│       ├── __init__.py
│       ├── __main__.py
│       ├── cli.py
│       └── langs/
│           └── english.py
└── hello-world-kannada/
    ├── setup.py
    └── hello_world_kannada/
        ├── __init__.py
        └── plugin.py
```
1. A core package hello-world that discovers language plugins
2. Aseparate plugin package hello-world-kannada that registers a Kannada plugin.

Core Packages

**hello-world/setup.py**
```python
from setuptools import setup, find_packages

setup(
    name="hello-world",
    version="0.1",
    packages=find_packages(),
    entry_points={
        # Register the built-in english plugin so it's discoverable like external plugins
        "hello_world.languages": [
            "english = hello_world.langs.english:EnglishHello",
        ],
        # optional convenience CLI script
        "console_scripts": [
            "hello-world=hello_world.cli:main",
        ],
    },
)
```

**hello-world/hello_world/__init__.py**
```python
__all__ = ["cli", "langs"]
__version__ = "0.1"
```

**hello-world/hello_world/langs/english.py**
```python
class EnglishHello:
    def hello(self):
        return "Hello, World!"
```

**hello-world/hello_world/cli.py**
```python
# hello_world/cli.py
import argparse
import sys
from importlib.metadata import entry_points


def load_plugins():
    """Return {name: class} mapping for plugins in group hello_world.languages"""
    eps = entry_points().select(group="hello_world.languages")
    return {ep.name: ep.load() for ep in eps}


def say_hello(language="english"):
    plugins = load_plugins()
    plugin_cls = plugins.get(language)
    if not plugin_cls:
        available = ", ".join(sorted(plugins.keys())) or "<none>"
        raise ValueError(f"Language '{language}' not supported. Available: {available}")
    return plugin_cls().hello()


def main(argv=None):
    parser = argparse.ArgumentParser(prog="hello-world")
    parser.add_argument("-l", "--language", default="english", help="language plugin to use")
    parser.add_argument("--list", action="store_true", help="list available plugins")
    args = parser.parse_args(argv)

    if args.list:
        plugins = sorted(load_plugins().keys())
        if plugins:
            print("Available plugins:")
            for p in plugins:
                print(" -", p)
        else:
            print("No plugins found.")
        return 0

    try:
        print(say_hello(args.language))
    except ValueError as e:
        print(e, file=sys.stderr)
        return 1
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

**hello-world/hello_world/__main__.py**
```python
from .cli import main

if __name__ == "__main__":
    raise SystemExit(main())
```

---

Pluggin Package : hello-world-kannada

**hello-world-kannada/setup.py**
```python
from setuptools import setup, find_packages

setup(
    name="hello-world-kannada",
    version="0.1",
    packages=find_packages(),
    entry_points={
        "hello_world.languages": [
            "kannada = hello_world_kannada.plugin:KannadaHello",
        ],
    },
)
```

**hello-world-kannada/hello_world_kannada/__init__.py**
```python
# Package Manager
```

**hello-world-kannada/hello_world_kannada/plugin.py**
```python
# -*- coding: utf-8 -*-
class KannadaHello:
    def hello(self):
        # Kannada: "Namaskara Prapancha" — Hello, World!
        return "ನಮಸ್ಕಾರ ಪ್ರಪಂಚ!"
```

---

**Installation**
```bash
python -m venv .venv
source .venv/bin/activate

# Install core package in editable mode
cd hello-world
pip install -e .
cd ..

# Check that the core CLI is available
# You can run via module (no PATH issues): python -m hello_world
python -m hello_world --list
# Expected output:
# Available plugins:
#  - english

# Install the Kannada plugin
cd hello-world-kannada
pip install -e .
cd ..

python -m hello_world --list
# Expected:
# Available plugins:
#  - english
#  - kannada

# Run English (builtin)
python -m hello_world --language english
# Output: Hello, World!

# Run Kannada (plugin)
python -m hello_world --language kannada
# Output: ನಮಸ್ಕಾರ ಪ್ರಪಂಚ!
```
