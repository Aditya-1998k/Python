# AST (Abstract Syntax Tree)
1. `AST` is the tree representation of source code.
2. Each node of the tree represents a `construct` in the code (like a **function**, **assignment**, **loop** or **variable**)
3. Python provides the built-in `ast` module to parse, analyze, and transform code dynamically.

---

### Why to use AST?
1. **Code Analysis** : Tools like linters (eg: `Flake8`, `Pylint`) use AST to analyze code without executing it.
2. **Code Transformation**: Refactoring tools like (`black formattor`, `autoPep8`) rely on AST to change code structure while
keeping behaviour intact.
3. **Security**: Safely inspect user-supplied code before running it.
4. **Custom Compilers/DSLs** : Build your own query language, rule engine or ORMs etc
- SQLAlchemy is a perfect real-world example of a DSL + compiler built in Python.
- DSL (Domain Specific Language) : You don’t write raw SQL. Instead, you write Python expressions that look like SQL

ASCII Diagram
```python
Python DSL (ORM/Query API)

users = Table("users", metadata, Column("id", Integer), Column("name", String))
query = select(users).where(users.c.id > 5)
        │
        ▼
  sqlalchemy turns your expression
  into expression tree (like python AST)
  Expression Tree (AST-like)

  Select
 └── From: users
 └── Where: BinaryExpression
              ├── left: users.id
              ├── op: ">"
              └── right: 5

        │
        ▼
  SQLAlchemy Compiler
  1. SQLAlchemy has compilers for different databases (Postgres, MySQL, MSSQL).
  2. It walks the expression tree and generates vendor-specific SQL
  3. If you switch DBs, SQLAlchemy compiles differently but the Python DSL stays the same.
        │
        ▼
   SQL String ("SELECT ...")
        │
        ▼
 Finally, SQLAlchemy sends the compiled SQL to the DB engine.
 Database Execution Engine
        │
        ▼
   Python Result Objects
```

