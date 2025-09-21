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

---

### Basic AST Parsing
```python
import ast

code = "x = 5 + 3"

# Parse into AST
tree = ast.parse(code)

# Pretty-print AST
print(ast.dump(tree, indent=4))
```
Output:
```
Module(
    body=[
        Assign(
            targets=[ 
                Name(id='x', ctx=Store())],
                value=BinOp(
                     left=Constant(value=5),
                     op=Add(),
                     right=Constant(value=3)
                    )
              )
    ],
    type_ignores=[]
)
```
1. **Module(...)** :
- Root Node of any python files/script.
- Its `.body` holdsa list of statements.
- Here, it has one statement: an `Assign`.

2. **Assign**:
- Represents an assignment like a = b.
- Has two parts:
1. targets → what we are assigning to.
2. value → what we are assigning.

3. **targets=[Name(id='x', ctx=Store())]**
- The left-hand side of = is a variable name: x.
-  ctx=Store() means x is in assignment context (we’re storing a value into it).
-   (If it was print(x), then it would be ctx=Load() — reading the variable).

3. **value=BinOp(...)**
Right-hand side is a binary operation: 5 + 3.

Inside:
- left=Constant(value=5) → number literal 5.
- op=Add() → the + operator.
- right=Constant(value=3) → number literal 3.

```bash
Module
 └── Assign
      ├── Target: Name(id="x", ctx=Store)
      └── Value: BinOp
           ├── Left: Constant(5)
           ├── Op: Add
           └── Right: Constant(3)
```

Execution Meaning
1. Create constant 5.
2. Create constant 3.
3. Apply binary operator +.
4. Store result (8) into variable x.

So at runtime, Python executes this AST as if you wrote:
```python
x = 5 + 3
```

### Traversing AST
You can walk through the Tree:
```python
for node in ast.walk(tree):
    print(type(node).__dict__)
```
Output:
```
Module
Assign
Name
BinOp
Constant
Add
Constant
Store
```

### Modifying Code Dynamically
Example: Transform a + b into a - b

```python
import ast

class ReplaceAddWithSubs(ast.NodeTransformer):
    def visit_BinOp(self, node):
        if isinstance(node.op, ast.Add):
            node.op = ast.Sub()
        return self.generic_visit(node)

code = "result = 10 + 5"
tree = ast.parse(code)
tree = ReplaceAddWithSubs().visit(tree)

# Compile modified AST back to Code
compiled = compile(tree, filename="<ast>", mode="exec")
exec(compiled)

print(result)
```
Output:
```
5
```

**Let's Decode Above code:**
##### 1. Parse the code in AST
```python
code = "result = 10 + 5"
tree = ast.parse(code)
```
AST Looks like
```python
Module
 └── Assign
      ├── Target: Name("result", ctx=Store)
      └── Value: BinOp
           ├── Left: Constant(10)
           ├── Op: Add
           └── Right: Constant(5)
```

##### 2. Define a Transformer
```python
class ReplaceAddWithSub(ast.NodeTransformer):
    def visit_BinOp(self, node):
        if isinstance(node.op, ast.Add):
            node.op = ast.Sub()
        return self.generic_visit(node)
```
- `NodeTransformer` lets you modify nodes in the AST.
- `visit_BinOp` is called for every `BinOp` node (binary operations like +, -, *, /).
- If the operator is Add, replace it with Sub.
- self.generic_visit(node) ensures child nodes are still visited.
So "10 + 5" becomes "10 - 5"

##### 3. Apply Transformer
```python
tree = ReplaceAddWithSub().visit(tree)
```
Now ast modified with:
```python
Module
 └── Assign
      ├── Target: Name("result", ctx=Store)
      └── Value: BinOp
           ├── Left: Constant(10)
           ├── Op: Sub   ← changed from Add → Sub
           └── Right: Constant(5)
```

##### 4. Compile & Execute
```python
compiled = compile(tree, filename="<ast>", mode="exec")
exec(compiled)
```

- `compile` takes the AST and turns it into executable Python bytecode.
- `exec` runs it.
- At runtime, it executes:
  ```python
  result = 10 - 5
  ```

  ASCII Flow
  ```python
   Source Code: "result = 10 + 5"
        │
        ▼
   ast.parse
        │
        ▼
  AST with BinOp(Add)
        │
        ▼
ReplaceAddWithSub (NodeTransformer)
        │
        ▼
  AST with BinOp(Sub)
        │
        ▼
   compile → exec
        │
        ▼
 Python executes: result = 10 - 5
        │
        ▼
   Output: 5
  ```
