# 使用FastAPI构建高性能API：从入门到实战

## 引言：为什么选择FastAPI？

Python的Web框架世界百花齐放，从重量级的Django到极简的Flask，各有千秋。然而，近年来，一个名为 **FastAPI** 的新星迅速崛起，成为构建API，特别是高性能API的首选框架之一。

FastAPI的核心优势在于其三大特性：
1.  **Fast (快速)**：基于Starlette（一个轻量级的ASGI框架）和Pydantic（一个数据验证库），FastAPI的性能与NodeJS和Go不相上下，是Python Web框架中性能最顶尖的之一。
2.  **Easy to Code (易于编码)**：它大量使用Python 3.7+的 **类型提示（Type Hints）**，不仅让代码更健壮、更易于维护，还带来了无与伦比的编辑器支持（如自动补全）和极低的开发成本。
3.  **Automatic Docs (自动文档)**：FastAPI能根据你的代码自动生成交互式的API文档（基于OpenAPI和JSON Schema），这意味着你无需再手动编写和维护API文档。

本指南将带你从零开始，快速掌握使用FastAPI构建现代API的核心技能。

---

## 快速入门：你的第一个API

在开始之前，请确保你已经安装了FastAPI和其所需的服务器Uvicorn：
```bash
pip install fastapi "uvicorn[standard]"
```

现在，创建一个名为 `main.py` 的文件，并写入以下代码：

```python
from fastapi import FastAPI

# 1. 创建一个FastAPI实例
app = FastAPI()

# 2. 定义一个路径操作装饰器
@app.get("/")
def read_root():
    # 3. 返回一个JSON响应
    return {"Hello": "World"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str | None = None):
    return {"item_id": item_id, "q": q}
```

**代码解析**：
1.  我们导入并创建了一个 `FastAPI` 应用实例。
2.  `@app.get("/")` 是一个 **路径操作装饰器**，它告诉FastAPI：当有客户端向根路径 `/` 发送 `GET` 请求时，由下面的 `read_root` 函数来处理。
3.  `read_item` 函数展示了FastAPI的两个强大功能：
    -   **路径参数**：`item_id: int` 表示从路径中捕获`item_id`，并将其转换为整数。
    -   **查询参数**：`q: str | None = None` 表示`q`是一个可选的字符串查询参数。

**运行它！**
在终端中运行以下命令：
```bash
uvicorn main:app --reload
```
`--reload` 参数会在你修改代码后自动重启服务器，非常适合开发环境。

---

## 自动交互式API文档

现在，打开你的浏览器，访问 `http://127.0.0.1:8000/docs`。

你会看到一个由Swagger UI生成的、功能齐全的交互式API文档。你可以在这里看到所有的API端点、参数、请求体，并可以直接在页面上进行API测试。这完全是FastAPI根据你的代码自动生成的！

你也可以访问 `http://127.0.0.1:8000/redoc` 查看另一种风格的文档。

---

## 核心概念：Pydantic与依赖注入

### 1. 使用Pydantic进行数据验证

FastAPI的魔力很大一部分来自于 **Pydantic**。它允许你使用标准的Python类型提示来定义数据模型，FastAPI会用它来做：
-   **数据校验**：自动验证请求数据的类型和约束。
-   **数据转换**：将请求数据转换为你定义的Python对象。
-   **数据文档**：自动在API文档中生成数据模型。

**示例：处理请求体（Request Body）**

```python
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.post("/items/")
async def create_item(item: Item):
    return item
```
在这个例子中：
-   我们定义了一个继承自 `pydantic.BaseModel` 的 `Item` 类。
-   在 `create_item` 函数中，我们将参数 `item` 的类型提示为 `Item`。
-   当一个POST请求发到 `/items/` 时，FastAPI会自动：
    1.  读取请求体（Request Body）。
    2.  校验其是否符合 `Item` 模型的结构和类型。
    3.  如果校验通过，则将JSON数据转换为一个 `Item` 类的实例，并传递给 `item` 参数。
    4.  如果校验失败，则自动返回一个包含清晰错误信息的422 Unprocessable Entity响应。

### 2. 依赖注入系统 (Dependency Injection)

FastAPI拥有一个非常强大且易于使用的依赖注入系统。它允许你将代码中的公共逻辑（如数据库连接、用户认证）封装成可复用的 **依赖项（Dependencies）**。

**示例：获取数据库连接**

```python
from fastapi import FastAPI, Depends

# 想象这是一个获取数据库会话的函数
def get_db_session():
    db = SessionLocal() # 创建会话
    try:
        yield db
    finally:
        db.close() # 关闭会话

app = FastAPI()

@app.get("/users/")
def read_users(db: Session = Depends(get_db_session)):
    # 在这里，db就是一个可用的数据库会话
    users = db.query(User).all()
    return users
```
通过 `Depends(get_db_session)`，FastAPI会在处理 `read_users` 请求前，先调用 `get_db_session` 函数，并将其返回（或 `yield`）的值注入到 `db` 参数中。这使得代码逻辑非常清晰，且易于测试。

---

## 异步支持：`async` / `await`

FastAPI是一个异步框架，这意味着它可以非常高效地处理I/O密集型任务（如等待数据库响应、调用外部API）。你可以自由地在路径操作函数中使用 `async def` 语法。

```python
import asyncio

@app.get("/")
async def read_results():
    # 假设这是一个耗时的I/O操作
    await asyncio.sleep(3)
    return [{"item_id": "Foo"}]
```
当FastAPI遇到一个 `async def` 函数时，它会以异步方式运行它。对于普通的 `def` 函数，它会将其在一个外部线程池中运行，以确保不会阻塞主事件循环。这使得你可以混合使用同步和异步代码，而无需担心性能问题。

## 结论

FastAPI通过巧妙地结合Python类型提示、Pydantic和Starlette，为现代API开发提供了一个兼具 **极高性能** 和 **卓越开发体验** 的解决方案。其自动文档、强大的数据验证和依赖注入系统，能够极大地提升开发效率和代码质量。

如果你正在开始一个新的Python API项目，或者希望为现有项目带来性能和开发效率的飞跃，FastAPI绝对是你最值得考虑的选择之一。