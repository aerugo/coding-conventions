# Python Style Guide for LLM Code Generation (Python 3.12 and Pydantic 2.0)

## Environment

- **Target Python Version**: **3.12**
- **Target Pydantic Version**: **2.0**

## General Principles

1. **Use Pydantic models as the primary data structure.**
2. **Prefer composition over inheritance.**
3. **Use Protocols to define interfaces instead of Abstract Base Classes (ABCs).**
4. **Use type hints consistently throughout the code.**
5. **Follow PEP 8 guidelines for naming conventions and code formatting.**

## Design Principles

### 1. High Cohesion and Low Coupling

- **High Cohesion**: Ensure each module or class has a single, well-defined responsibility.
- **Low Coupling**: Minimize dependencies between modules and classes using dependency injection and well-defined interfaces.

### 2. Favor Composition Over Inheritance

- **Use composition** to build complex objects from simpler ones.
- **Avoid deep inheritance hierarchies**.
- **Use mixins sparingly**.

### 3. Start with the Data

- **Define data structures first** using Pydantic models.
- **Use type hints** to clearly specify data types.
- **Design APIs around data structures**, not behaviors.

### 4. Depend on Abstractions

- **Use Protocols** to define interfaces.
- **Code against interfaces (Protocols)** rather than concrete implementations.
- **Apply dependency inversion** to decouple modules.

### 5. Separate Creation from Use

- **Implement factory patterns** or **use dependency injection** for object creation.
- **Pass dependencies as parameters**; avoid creating them within methods.
- **Use configuration objects** for complex object creation.

### 6. Keep Things Simple

- **Prefer simple solutions** over complex ones.
- **Avoid unnecessary functionality** (YAGNI principle).
- **Refactor and simplify code regularly**.

## Coding Standards

### Type Hints

1. **Use built-in generic types**: `list[int]`, `dict[str, float]`, `tuple[int, int]`.
2. **Import types from `typing`** when necessary: `Callable`, `TypeVar`, `Generic`, `Protocol`.
3. **Use `|` for union types**: `int | str`.
4. **Use `Type | None`** for optional types.
5. **Annotate all function parameters and return types**.
6. **Handle forward references** with postponed evaluation of annotations (default in Python 3.12).

**Example:**

```python
def process_data(data: list[int | str]) -> dict[str, int]:
    ...
```

### Protocols and Interfaces

1. **Use `Protocol`** from `typing` to define interfaces.
2. **Name Protocols clearly**, optionally using a "Protocol" suffix.
3. **Define methods in Protocols** without implementations.
4. **Use structural subtyping** (duck typing).
5. **Use `@runtime_checkable`** if `isinstance()` checks are needed.

**Example:**

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class DataProcessor(Protocol):
    def process(self, data: list[int]) -> list[int]:
        ...
```

### Enums

1. **Use `UpperCamelCase`** for enum class names, suffixed with `Enum`.
2. **Use `UPPER_SNAKE_CASE`** for enum members.
3. **Assign values** using integers, strings, or `auto()`.
4. **Use `==` operator** for comparisons.

**Example:**

```python
from enum import Enum, auto

class StatusEnum(Enum):
    SUCCESS = auto()
    FAILURE = auto()

if result.status == StatusEnum.SUCCESS:
    ...
```

## Data Modeling

### Pydantic Models (Pydantic 2.0)

1. **Inherit from `pydantic.BaseModel`**.
2. **Use type annotations** for all fields.
3. **Use `Field()`** for validation and metadata.
4. **Use `@field_validator`** for custom validation logic.
5. **Use the `model_config`** attribute for model-wide settings.
6. **Use Pydantic's built-in types**, like `EmailStr`.
7. **Leverage Pydantic V2 features**, such as `model_serializers` and improved validation.

**Example:**

```python
from pydantic import BaseModel, Field, EmailStr, field_validator

class User(BaseModel):
    id: int
    name: str = Field(..., max_length=50)
    email: EmailStr

    @field_validator('name')
    def name_must_be_capitalized(cls, value: str) -> str:
        if not value.istitle():
            raise ValueError('Name must be capitalized')
        return value

    model_config = {
        'json_schema_extra': {
            'examples': [
                {
                    'id': 1,
                    'name': 'John Doe',
                    'email': 'john.doe@example.com'
                }
            ]
        }
    }
```

### Composition Patterns

1. **Compose models** by using other models as fields.
2. **Create models** for shared attributes.
3. **Use functions or services** for shared behavior.
4. **Keep Pydantic and ORM models in sync**, defining mappings between them.

**Example:**

```python
class Address(BaseModel):
    street: str
    city: str
    zip_code: str

class Customer(BaseModel):
    id: int
    name: str
    address: Address
```

## Application Structure

### API Layer

1. **Use Pydantic models** for request and response schemas.
2. **Use FastAPI** for API development.
3. **Design RESTful endpoints** around resources.
4. **Use `async def`** for I/O-bound endpoints.
5. **Use `Depends`** for dependency injection.
6. **Validate inputs** with Pydantic.

**Example:**

```python
from fastapi import FastAPI, Depends
from pydantic import BaseModel

app = FastAPI()

class ProductCreate(BaseModel):
    name: str
    price: float

class ProductResponse(BaseModel):
    id: int
    name: str
    price: float

@app.post("/products/", response_model=ProductResponse)
async def create_product(product: ProductCreate):
    ...
```

### Database Layer

1. **Use SQLAlchemy** for ORM.
2. **Create separate ORM and Pydantic models**, keeping them in sync.
3. **Link models** via composition.
4. **Use the repository pattern** with Protocols.
5. **Use async drivers** for database operations.
6. **Leverage Python 3.12 features** for improved performance.
7. **Use Alembic** for database schema migrations.

**SQLAlchemy ORM Example:**

```python
# ORM model using SQLAlchemy
from sqlalchemy import Column, Integer, String, Float
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.ext.asyncio import AsyncSession

Base = declarative_base()

class ProductORM(Base):
    __tablename__ = 'products'
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(255), nullable=False)
    price = Column(Float, nullable=False)

# Pydantic model
from pydantic import BaseModel

class Product(BaseModel):
    id: int
    name: str
    price: float

    @classmethod
    def from_orm(cls, orm_obj: ProductORM) -> 'Product':
        return cls(
            id=orm_obj.id,
            name=orm_obj.name,
            price=orm_obj.price,
        )
```

**Using Alembic for Migrations:**

1. **Initialize Alembic** in your project using `alembic init alembic`.
2. **Configure `alembic.ini` and `env.py`** to connect to your database and import your models.
3. **Generate migration scripts** using `alembic revision --autogenerate -m "create products table"`.
4. **Apply migrations** using `alembic upgrade head`.

**Alembic Configuration Example:**

```ini
# alembic.ini
[alembic]
script_location = alembic
sqlalchemy.url = sqlite+aiosqlite:///./test.db
```

```python
# alembic/env.py
from logging.config import fileConfig
from sqlalchemy import engine_from_config
from sqlalchemy import pool
from alembic import context
import your_app.models  # Import your ORM models here

config = context.config
fileConfig(config.config_file_name)
target_metadata = your_app.models.Base.metadata  # Use your Base's metadata

def run_migrations_online():
    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix='sqlalchemy.',
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(connection=connection, target_metadata=target_metadata)

        with context.begin_transaction():
            context.run_migrations()

run_migrations_online()
```

### Dependency Management

1. **Use dependency injection** for classes and functions.
2. **Implement a container** or use libraries like `dependency-injector`.
3. **Define interfaces** with Protocols.
4. **Use `Depends`** in FastAPI endpoints.

**Example:**

```python
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker

DATABASE_URL = "sqlite+aiosqlite:///./test.db"

engine = create_async_engine(DATABASE_URL)
async_session = async_sessionmaker(engine, expire_on_commit=False)

async def get_db() -> AsyncSession:
    async with async_session() as session:
        yield session

@app.get("/products/")
async def read_products(db: AsyncSession = Depends(get_db)):
    ...
```

### Asynchronous Programming

1. **Use `async def`** for I/O-bound functions.
2. **Use async libraries** for database and network operations.
3. **Avoid blocking calls** in async functions.
4. **Utilize `asyncio` improvements** in Python 3.12, like Task Groups.

**Example:**

```python
import asyncio

async def fetch_data():
    ...

async def process_data():
    ...

async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(fetch_data())
        tg.create_task(process_data())
```

## Best Practices

### Testing

1. **Use pytest** for tests.
2. **Mirror application structure** in tests.
3. **Use fixtures** for setup/teardown.
4. **Focus on meaningful tests**.
5. **Use mocks** to isolate tests.
6. **Test async code** with `pytest-asyncio`.
7. **Utilize new testing features** in Python 3.12.

**Example:**

```python
import pytest
import asyncio

@pytest.mark.asyncio
async def test_create_product():
    ...
```

### Security

1. **Validate all inputs** with Pydantic.
2. **Sanitize outputs** to prevent injections.
3. **Implement authentication and authorization**.
4. **Securely handle sensitive data**.
5. **Use type annotations** to prevent type-related vulnerabilities.

### Code Readability

1. **Use descriptive names** for all identifiers.
2. **Adhere to the Single Responsibility Principle**.
3. **Group related code** logically.
4. **Use type aliases** for complex types.
5. **Create functions for shared behavior**.

### Functional Programming

1. **Write pure functions** when possible.
2. **Avoid global state**.
3. **Use immutable data structures**.
4. **Leverage `functools` and `itertools`** for functional patterns.

### When to Use Regular Classes

1. **For complex behavior** implementations.
2. **For performance-critical code**.
3. **When integrating with external libraries** that require inheritance.

## Example Structures

### Protocol Definition

```python
from typing import Protocol, runtime_checkable
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str

@runtime_checkable
class UserRepository(Protocol):
    async def get_user(self, user_id: int) -> User:
        ...

    async def create_user(self, user: User) -> User:
        ...
```

### Composition Example

```python
from pydantic import BaseModel
from datetime import datetime

class Timestamps(BaseModel):
    created_at: datetime
    updated_at: datetime

class Post(BaseModel):
    id: int
    title: str
    content: str
    timestamps: Timestamps
```

### Dependency Injection Example

```python
from typing import Protocol
from pydantic import BaseModel

class NotificationService(Protocol):
    async def send_notification(self, message: str) -> None:
        ...

class AlertSystem:
    def __init__(self, notifier: NotificationService):
        self.notifier = notifier

    async def alert(self, msg: str):
        await self.notifier.send_notification(msg)
```

### API Endpoint with Dependency Injection

```python
from fastapi import FastAPI, Depends
from pydantic import BaseModel
from typing import Protocol

app = FastAPI()

class MessageCreate(BaseModel):
    content: str

class MessageResponse(BaseModel):
    id: int
    content: str

class MessageService(Protocol):
    async def create_message(self, message: MessageCreate) -> MessageResponse:
        ...

async def get_message_service() -> MessageService:
    ...

@app.post("/messages/", response_model=MessageResponse)
async def create_message(
    message: MessageCreate,
    service: MessageService = Depends(get_message_service)
):
    return await service.create_message(message)
```