---
parent: Decisions
nav_order: 34
title: 0034 Data Validation
tags:
  - python
  - validation
  - pydantic
  - schema
  - types
  - data
  - configuration
---
# Data Validation with Pydantic

## Context and Problem Statement

Reliable applications require thorough validation of input data to ensure integrity, prevent security vulnerabilities, and provide clear error messages when data doesn't conform to expected formats. Additionally, application configurations need type safety and validation. How should we standardize data validation across our Python applications?

## Considered Options

* Manual validation with custom functions and conditionals
* marshmallow for object serialization/deserialization
* dataclasses with type annotations and custom validators
* cerberus validation library
* jsonschema for JSON validation
* Pydantic for data validation and settings management

## Decision Outcome

Chosen option: **Pydantic for data validation and settings management**, because:

* It provides a simple, intuitive interface using Python type annotations
* It integrates seamlessly with FastAPI and works well with Flask
* It offers comprehensive validation capabilities with clear error messages
* It handles complex nested data structures efficiently
* It includes built-in support for environment variable loading and configuration management
* It supports JSON Schema generation for API documentation

## Details

### Core Principles

1. **Validate at boundaries** - All data entering the application from external sources (API requests, file imports, third-party integrations) must be validated.

2. **Fail early** - Validation should occur as early as possible in the request lifecycle to prevent invalid data from propagating through the system.

3. **Clear error messages** - Validation errors should provide clear, actionable information about what failed and why.

4. **Type safety** - Use Python type annotations with Pydantic models to ensure type safety and IDE support.

5. **Documentation as code** - Pydantic models serve as both validation and documentation of data structures.

### Implementation Guidelines

#### Basic Data Models

Define Pydantic models for all data structures that require validation:

```python
from datetime import datetime
from typing import List, Optional
from pydantic import BaseModel, Field, EmailStr, HttpUrl, constr

class User(BaseModel):
    id: Optional[int] = None
    username: constr(min_length=3, max_length=50)
    email: EmailStr
    full_name: str
    disabled: bool = False
    created_at: datetime = Field(default_factory=datetime.utcnow)
    
    class Config:
        populate_by_name = True
```

#### Request/Response Models

Create separate models for API requests and responses:

```python
class UserCreateRequest(BaseModel):
    username: constr(min_length=3, max_length=50)
    email: EmailStr
    full_name: str
    password: constr(min_length=8)

class UserResponse(BaseModel):
    id: int
    username: str
    email: EmailStr
    full_name: str
    disabled: bool
    created_at: datetime
```

#### Configuration Management

Use Pydantic's settings management for application configuration:

```python
from pydantic import BaseSettings, Field, PostgresDsn, SecretStr

class Settings(BaseSettings):
    APP_NAME: str = "MyApp"
    DEBUG: bool = False
    DATABASE_URL: PostgresDsn
    SECRET_KEY: SecretStr
    API_KEY: SecretStr
    LOG_LEVEL: str = "INFO"
    
    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"
        case_sensitive = True
```

#### Integration with Web Frameworks

For **FastAPI**, Pydantic models are used directly for request validation:

```python
from fastapi import FastAPI, Depends, HTTPException

app = FastAPI()

@app.post("/users/", response_model=UserResponse)
async def create_user(user: UserCreateRequest):
    # The input is already validated by Pydantic
    db_user = create_user_in_db(user)
    return db_user
```

For **Flask**, use Pydantic models explicitly:

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/users/", methods=["POST"])
def create_user():
    try:
        # Validate request data
        user_data = UserCreateRequest(**request.json)
        db_user = create_user_in_db(user_data)
        # Serialize response
        return jsonify(UserResponse.from_orm(db_user).dict())
    except ValidationError as e:
        return jsonify({"error": e.errors()}), 400
```

#### SQLAlchemy Integration

When using Pydantic with SQLAlchemy:

```python
from sqlalchemy import Column, Integer, String, Boolean, DateTime
from sqlalchemy.ext.declarative import declarative_base
from pydantic import BaseModel
from datetime import datetime

Base = declarative_base()

# SQLAlchemy model
class UserDB(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    username = Column(String, unique=True, index=True)
    email = Column(String, unique=True, index=True)
    full_name = Column(String)
    hashed_password = Column(String)
    disabled = Column(Boolean, default=False)
    created_at = Column(DateTime, default=datetime.utcnow)

# Pydantic model for ORM
class UserModel(BaseModel):
    id: Optional[int] = None
    username: str
    email: str
    full_name: str
    disabled: bool = False
    created_at: datetime
    
    class Config:
        orm_mode = True  # Enables reading data from ORM objects
```

#### Validation Customization

Use custom validators for complex validation logic:

```python
from pydantic import BaseModel, validator

class Transaction(BaseModel):
    amount: float
    currency: str
    description: str
    
    @validator('amount')
    def amount_must_be_positive(cls, v):
        if v <= 0:
            raise ValueError('amount must be positive')
        return v
    
    @validator('currency')
    def currency_must_be_valid(cls, v):
        valid_currencies = ['USD', 'EUR', 'GBP', 'JPY']
        if v not in valid_currencies:
            raise ValueError(f'currency must be one of {valid_currencies}')
        return v
```

### Best Practices

1. **Keep models focused** - Each model should have a single responsibility and represent a specific use case.

2. **Separate domain models from API models** - Use different models for internal domain logic and API interfaces.

3. **Use validation strictly** - Enable `strict=True` in Pydantic models to enforce stricter type checking.

4. **Leverage type annotations** - Use Python's type annotations to their full extent for better documentation and IDE support.

5. **Define clear constraints** - Use Field constraints to clearly define validation rules:

```python
from pydantic import BaseModel, Field

class Product(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    price: float = Field(..., gt=0)
    description: Optional[str] = Field(None, max_length=1000)
    tags: List[str] = Field(default_factory=list, max_items=10)
```

6. **Document models** - Use docstrings and Field descriptions to document model fields:

```python
class User(BaseModel):
    """
    Represents a user in the system.
    """
    username: str = Field(
        ..., 
        description="Unique username between 3-50 characters",
        min_length=3, 
        max_length=50
    )
```

## Consequences

### Positive

* Improved data validation at application boundaries
* Clear, actionable error messages for invalid data
* Self-documenting data structures through type annotations
* Reduced boilerplate code for validation logic
* Strong integration with FastAPI and good support for Flask
* Type safety throughout the application

### Negative

* Learning curve for developers unfamiliar with Pydantic
* Minor performance overhead compared to manual validation (though usually negligible)
* Potential complexity in deeply nested validation scenarios

## Related Decisions

* [0011 Language and Framework](./0011-language-and-framework.md) - Python as the primary language with type annotations
* [0018 Python Standards](./0018-python-standards.md) - Code style and typing standards
* [0032 Web Framework Selection](./0032-web-framework-selection.md) - Flask and FastAPI framework standards

## Notes

While this ADR standardizes on Pydantic for data validation, it's important to recognize that Pydantic v2 (released in 2023) introduced significant changes and performance improvements. **Projects should use the latest stable version of Pydantic to benefit from these improvements.**

For very performance-critical applications where validation speed is crucial, consider using Pydantic with the optional Rust-based validator for even faster validation.
