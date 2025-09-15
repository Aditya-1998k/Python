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

**Core Packages**
```
hello-world/
├── setup.py
└── hello_world/
    ├── __init__.py
    ├── app.py
    └── langs/
        ├── __init__.py
        └── english.py

```
1. A core package hello-world that discovers language plugins
2. Aseparate plugin package hello-world-kannada that registers a Kannada plugin.

Core Packages

**hello-world/setup.py**
```python
from setuptools import setup

setup(
    name="hello-world",
    version="0.1",
    packages=["hello_world", "hello_world.langs"],
    entry_points={
        "hello_world.languages": [
            "english = hello_world.langs.english:EnglishHello",
        ],
    },
)
```

**hello-world/lang/english.py**
```python
class EnglishHello:
    def hello(self):
        return "Hello, World!"
```

**hello-world/hello_world/langs/english.py**
```python
class EnglishHello:
    def hello(self):
        return "Hello, World!"
```

**hello-world/app.py**
```python
from importlib.metadata import entry_points


def load_plugins():
    """Discover plugins registered under hello_world.languages"""
    return {
        ep.name: ep.load()
        for ep in entry_points().select(group="hello_world.languages")
    }

def say_hello(language="english"):
    """Run the hello method of the selected plugin"""
    plugins = load_plugins()
    if language not in plugins:
        raise ValueError(f"Language '{language}' not supported")
    return plugins[language]().hello()


if __name__ == "__main__":
    print(say_hello("english"))
```

---

Pluggin Package : hello-world-kannada
```
hello-world-kannada/
├── setup.py
└── hello_world_kannada/
    ├── __init__.py
    └── plugin.py
```

**hello-world-kannada/setup.py**
```python
from setuptools import setup

setup(
    name="hello-world-kannada",
    version="0.1",
    packages=["hello_world_kannada"],
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

# Uninstall
pip uninstall hello-world hello-world-kannada
```

Other Note:
1. pip install -e . makes an editable install so you can change files and re-run without reinstalling.
2. If you prefer non-editable installs, use pip install ./hello-world and pip install ./hello-world-kannada

**Explaination:**
1. Both packages register an entry point under the same group hello_world.languages.
2. The core package uses importlib.metadata.entry_points(...) to discover installed plugins at runtime.
3. Each entry point points to a Python object (here a class) which the core loads with .load() and uses uniformly (cls().hello()).
4. Plugins can be shipped independently and installed later — no change needed in core.

