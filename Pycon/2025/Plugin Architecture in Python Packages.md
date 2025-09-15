## Understanding Plugin Architecture for Python Packages

#### **What and Why Plugin Architecture?**
Plugin = Extension mechanism: A way to add new functionality without modifying the core package.

#### Why plugins?
1. Keeps core codebase small and maintainable.
2. Allows 3rd parties to extend functionality.
3. Widely used in tools like SQLAlchemy (dialects), Airflow (providers), Flask (extensions), pytest (plugins).

Sometimes, you want to make your program extensible without editing the core code.

**Example:**

- SQLAlchemy supports many databases (Postgres, MySQL, SQLite, …).
- It does this by loading “dialects” as plugins dynamically.
In Python, this is done using entry points declared in setup.py (or pyproject.toml).

Note: With plugins, new features can be added later just by installing another package.

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

#### **Core Packages**
This is your main package that knows:
- How to load plugins.
- How to call them.
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
- Defines a plugin group: hello_world.languages.
- Registers "english" plugin → loads EnglishHello class from hello_world.langs.english.

**hello-world/lang/english.py**
```python
class EnglishHello:
    def hello(self):
        return "Hello, World!"
```
A very simple plugin class — it just says hello in English.


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
What’s happening here:
1. `entry_points().select(group="hello_world.languages")`: finds all installed plugins in that group.
2. `ep.load()`: dynamically loads the class/function.
3. `say_hello("english")`: finds the right class (EnglishHello), creates an object, calls .hello().

---

#### Pluggin Package : hello-world-kannada
This is a separate package, which extends the base package.

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
Registers new plugin:
1. Group = hello_world.languages
2. Name = kannada
3. Target = hello_world_kannada.plugin:KannadaHello

**hello-world-kannada/pluggin.py**
```python
class KannadaHello:
    def hello(self):
        return "ನಮಸ್ಕಾರ ಪ್ರಪಂಚ!"
```
Adds Kannada greeting to the system.

---

**Installation**
```bash
python -m venv .venv
source .venv/bin/activate

# Install both packages
pip install -e ./hello-world
pip install -e ./hello-world-kannada

# Run in English (base package)
python -m hello_world.app
# Output
# Hello, World!
```

Call Kannada Plugging:

```python
from hello_world.app import say_hello
print(say_hello("kannada"))
```

Other Note:
1. pip install -e . makes an editable install so you can change files and re-run without reinstalling.
2. If you prefer non-editable installs, use pip install ./hello-world and pip install ./hello-world-kannada

**How it works (big picture)**
1. Base package defines a group (`hello_world.languages`) and provides loader functions.
2. Each plugin package registers itself in that group with an entry point.
3. When your app runs, `importlib.metadata` looks at installed packages and loads available plugins.
4. You can now call any plugin without knowing it in advance.

### importlib.metadata (What is this?)
- `importlib.metadata` is a standard library module (added in Python 3.8).
- It allows you to access metadata about installed Python packages, including:
1. version
2. entry points (plugins)
3. distribution details (author, license, etc.)
It’s basically Python’s built-in way to query installed packages (something pkg_resources from setuptools used to do, but lighter and faster).

##### **Why do we use it here?**
We’re using entry points to implement a plugin system.
- Packages declare entry points in setup.py or pyproject.toml.
- importlib.metadata.entry_points() lets us discover all registered plugins in a group.
```python
from importlib.metadata import entry_points

eps = entry_points().select(group="hello_world.languages")
for ep in eps:
    print(ep.name, "->", ep.value)
```
Output:
```
english -> hello_world.langs.english:EnglishHello
kannada -> hello_world_kannada.plugin:KannadaHello
```

##### What is an entry point object?
Each EntryPoint object has:
1. `.name` → the plugin’s name ("english", "kannada")
2. `.group` → the group it belongs to ("hello_world.languages")
3. `.value` → the dotted path string ("hello_world.langs.english:EnglishHello")
4. `.load()` → actually loads and returns the class/function

```python
ep.load()
```
Imports the class EnglishHello or KannadaHello dynamically

With importlib.metadata, the app doesn’t need to know about new languages —
they are just “discovered” at runtime!

**In shorts:**
`importlib.metadata` is Python’s way of letting packages “announce” themselves to other packages.
It is the backbone of Python’s plugin systems (like `SQLAlchemy dialects`, Airflow providers, `Flask extensions`, etc.).
