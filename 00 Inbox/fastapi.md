---
title: FastAPI Reference — Concepts, Patterns & Examples
tags: [fastapi, python, api, rest, asgi, pydantic, reference]
created: 2026-10-15
scope: Core concepts · Request/response handling · Pydantic v2 · Auth · Databases · Testing · Deployment · Full worked example
---

# FastAPI Reference

A practical reference for FastAPI: concepts, patterns, and **runnable examples** at every section, ending with a complete end-to-end project.

**Stack:** FastAPI is a thin layer over **Starlette** (ASGI toolkit, routing, middleware) and **Pydantic v2** (validation and serialisation). It speaks **ASGI**, so it runs on any ASGI server — uvicorn, hypercorn, granian, or gunicorn with an ASGI worker.

**Version notes:** Pydantic **v2** is required from FastAPI 0.100 onward; v1 code needs the `pydantic.v1` shim or a port. `Annotated` dependency style is the current recommendation (since 0.95) over default-value style. `@app.on_event` is deprecated in favour of lifespan handlers.

> **Verify before relying on it:** FastAPI moves quickly and the CLI (`fastapi dev`) and SSE helpers are recent additions. Run `pip show fastapi pydantic` to confirm what you actually have installed rather than trusting a version claim here.

---

## Table of Contents

- [[#1. What FastAPI Is|1. What FastAPI Is]]
- [[#2. Installation & Running|2. Installation & Running]]
    - [[#Running|Running]]
- [[#3. Minimal Application|3. Minimal Application]]
- [[#4. Path Operations & HTTP Methods|4. Path Operations & HTTP Methods]]
- [[#5. Path Parameters|5. Path Parameters]]
- [[#6. Query Parameters|6. Query Parameters]]
- [[#7. Request Bodies with Pydantic|7. Request Bodies with Pydantic]]
    - [[#Validation error shape|Validation error shape]]
- [[#8. Pydantic v2 Essentials|8. Pydantic v2 Essentials]]
- [[#9. Response Models|9. Response Models]]
- [[#10. Response Classes, Headers & Cookies|10. Response Classes, Headers & Cookies]]
- [[#11. Status Codes|11. Status Codes]]
- [[#12. Forms, Files & Uploads|12. Forms, Files & Uploads]]
- [[#13. Headers & Cookies (Input)|13. Headers & Cookies (Input)]]
- [[#14. Dependency Injection|14. Dependency Injection]]
    - [[#Yield dependencies (resource lifecycle)|Yield dependencies (resource lifecycle)]]
    - [[#Caching and reuse|Caching and reuse]]
- [[#15. Security & Authentication|15. Security & Authentication]]
    - [[#Password hashing|Password hashing]]
    - [[#Full JWT auth example|Full JWT auth example]]
    - [[#Scopes|Scopes]]
- [[#16. Middleware & CORS|16. Middleware & CORS]]
- [[#17. Routers & Project Structure|17. Routers & Project Structure]]
    - [[#Recommended layout (grows with the app)|Recommended layout (grows with the app)]]
- [[#18. Error Handling|18. Error Handling]]
- [[#19. Background Tasks|19. Background Tasks]]
- [[#20. Lifespan & Application State|20. Lifespan & Application State]]
- [[#21. Async vs Sync|21. Async vs Sync]]
- [[#22. Database Integration|22. Database Integration]]
    - [[#Async SQLAlchemy 2.0|Async SQLAlchemy 2.0]]
    - [[#Migrations (Alembic)|Migrations (Alembic)]]
- [[#23. Settings & Configuration|23. Settings & Configuration]]
- [[#24. Templates & Static Files|24. Templates & Static Files]]
- [[#25. WebSockets & Server-Sent Events|25. WebSockets & Server-Sent Events]]
    - [[#WebSockets|WebSockets]]
    - [[#Server-Sent Events|Server-Sent Events]]
- [[#26. Testing|26. Testing]]
    - [[#Sync with `TestClient`|Sync with `TestClient`]]
    - [[#Async with `httpx`|Async with `httpx`]]
    - [[#Overriding dependencies|Overriding dependencies]]
- [[#27. OpenAPI & Docs Customisation|27. OpenAPI & Docs Customisation]]
- [[#28. Deployment|28. Deployment]]
    - [[#Dockerfile|Dockerfile]]
    - [[#Production checklist|Production checklist]]
- [[#29. Performance|29. Performance]]
- [[#30. CLI Commands|30. CLI Commands]]
- [[#31. Common Pitfalls|31. Common Pitfalls]]
- [[#32. Full Worked Example|32. Full Worked Example]]
    - [[#Project layout|Project layout]]
    - [[#`app/config.py`|`app/config.py`]]
    - [[#`app/database.py`|`app/database.py`]]
    - [[#`app/models.py`|`app/models.py`]]
    - [[#`app/schemas.py`|`app/schemas.py`]]
    - [[#`app/security.py`|`app/security.py`]]
    - [[#`app/deps.py`|`app/deps.py`]]
    - [[#`app/routers/auth.py`|`app/routers/auth.py`]]
    - [[#`app/routers/items.py`|`app/routers/items.py`]]
    - [[#`app/main.py`|`app/main.py`]]
    - [[#`tests/conftest.py`|`tests/conftest.py`]]
    - [[#`tests/test_items.py`|`tests/test_items.py`]]
    - [[#`requirements.txt`|`requirements.txt`]]
    - [[#Run it|Run it]]
- [[#Appendix A: Pydantic v1 → v2 Cheat Sheet|Appendix A: Pydantic v1 → v2 Cheat Sheet]]
- [[#Appendix B: Dependency Injection Patterns|Appendix B: Dependency Injection Patterns]]
- [[#Appendix C: Response Payload Rules|Appendix C: Response Payload Rules]]
- [[#Appendix D: Quick Reference Card|Appendix D: Quick Reference Card]]

---


## 1. What FastAPI Is

| Component | Role |
|---|---|
| ASGI | The async server interface FastAPI implements |
| Starlette | Routing, middleware, responses, WebSockets, test client |
| Pydantic v2 | Request/response validation, parsing, serialisation |
| Uvicorn | The ASGI server you typically run in development |
| OpenAPI | Auto-generated schema served at `/openapi.json` |
| Swagger UI | Interactive docs at `/docs` |
| ReDoc | Alternative docs at `/redoc` |
| Dependency injection | A first-class system for shared logic and resources |
| Type hints | Drive validation, serialisation, and docs simultaneously |

**The core idea:** you declare types once, and FastAPI derives request parsing, validation, error responses, serialisation, and OpenAPI documentation from them. There is no separate schema to keep in sync.

| You write | FastAPI derives |
|---|---|
| `item_id: int` in the path | Integer parsing, 422 on failure, docs |
| `q: str \| None = None` | Optional query parameter |
| `item: Item` (a `BaseModel`) | JSON body parsing and validation |
| `response_model=ItemOut` | Response filtering and docs schema |
| `Depends(get_db)` | Injected resource with per-request caching |
| `status_code=201` | Documented status code |

## 2. Installation & Running

```bash
# Project setup
python3 -m venv .venv
source .venv/bin/activate

# Core
pip install "fastapi[standard]"

# Or the minimal set, choosing your own server
pip install fastapi uvicorn[standard]

# Common companions
pip install "pydantic-settings"        # settings from env/.env
pip install "python-multipart"         # Form() and File() support — REQUIRED for uploads
pip install "pyjwt" "pwdlib[argon2]"   # JWT auth and password hashing
pip install "sqlalchemy[asyncio]" asyncpg alembic
pip install "httpx" "pytest" "anyio"   # testing (anyio ships the pytest plugin)
```

`fastapi[standard]` pulls in uvicorn, the `fastapi` CLI, httpx, jinja2, python-multipart, and email-validator — the convenient default.

### Running

```bash
# Development: reload on change (fastapi CLI — this is the current recommended way)
fastapi dev main.py --port 8000

# Production
fastapi run main.py --workers 4

# Equivalent with uvicorn directly
uvicorn main:app --reload --host 127.0.0.1 --port 8000
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4

# Gunicorn managing ASGI workers (still common; needs `pip install uvicorn-worker`)
gunicorn main:app -k uvicorn_worker.UvicornWorker -w 4 -b 0.0.0.0:8000
```

`main:app` means "the `app` object inside `main.py`". Docs land at `http://127.0.0.1:8000/docs`.

## 3. Minimal Application

```python
# main.py
from fastapi import FastAPI

app = FastAPI(
    title="My API",
    version="1.0.0",
    description="A minimal FastAPI application.",
)


@app.get("/")
async def root():
    return {"message": "Hello World"}


@app.get("/health")
async def health():
    return {"status": "ok"}
```

Run it:

```bash
fastapi dev main.py
curl http://127.0.0.1:8000/
# {"message":"Hello World"}
```

Returned dicts and Pydantic models are serialised to JSON automatically. Lists, strings, numbers, and `None` all work; the content type is inferred.

## 4. Path Operations & HTTP Methods

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items")
async def list_items(): ...


@app.post("/items", status_code=201)
async def create_item(): ...


@app.put("/items/{item_id}")
async def replace_item(item_id: int): ...


@app.patch("/items/{item_id}")
async def update_item(item_id: int): ...


@app.delete("/items/{item_id}", status_code=204)
async def delete_item(item_id: int): ...


# Less common but available
@app.options("/items")
async def item_options(): ...


@app.head("/items")
async def item_head(): ...
```

| Decorator parameter | Purpose |
|---|---|
| `path` | The URL path template |
| `status_code` | Default success status |
| `response_model` | Serialisation/filtering schema |
| `tags` | Group in the docs |
| `summary` / `description` | Override docs text |
| `response_description` | Describe the success response |
| `responses` | Extra documented responses |
| `deprecated=True` | Mark as deprecated |
| `include_in_schema=False` | Hide from OpenAPI |
| `name` | Operation ID in the schema |
| `dependencies=[Depends(...)]` | Apply dependencies to this route only |

**Path matching is in declaration order.** A literal route declared after a parameterised one that matches it will be shadowed:

```python
@app.get("/items/me")          # must come FIRST
@app.get("/items/{item_id}")   # else "me" is captured as item_id
```

## 5. Path Parameters

```python
from enum import Enum
from fastapi import FastAPI, Path

app = FastAPI()


class Size(str, Enum):
    small = "small"
    medium = "medium"
    large = "large"


@app.get("/items/{item_id}")
async def get_item(item_id: int):
    # item_id arrives parsed, or the request 422s before your code runs
    return {"item_id": item_id}


@app.get("/sizes/{size}")
async def by_size(size: Size):
    # Only the three enum values are accepted
    return {"size": size}


# Validation via Path()
@app.get("/users/{user_id}")
async def get_user(
    user_id: int = Path(..., title="User ID", ge=1, le=1_000_000),
):
    return {"user_id": user_id}


# Annotated style — the current recommendation
from typing import Annotated

@app.get("/orders/{order_id}")
async def get_order(
    order_id: Annotated[int, Path(title="Order ID", ge=1)],
):
    return {"order_id": order_id}
```

| `Path(...)` constraint | Meaning |
|---|---|
| `gt` / `ge` | Greater than / greater-or-equal |
| `lt` / `le` | Less than / less-or-equal |
| `min_length` / `max_length` | String length bounds |
| `pattern` | Regex the value must match (Pydantic v2 name) |
| `title` / `description` | Docs metadata |
| `deprecated` | Mark deprecated |
| `examples` | Sample values in the docs |

Type conversion is automatic for `int`, `float`, `str`, `bool`, `UUID`, `datetime`, `date`, `Decimal`, `Enum`, and `Path`-typed segments. A bad value returns **422 Unprocessable Entity** with a structured error body.

## 6. Query Parameters

Any function parameter that is **not** in the path and is a scalar type becomes a query parameter. Parameters with defaults are optional; without defaults they are required.

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

FAKE = [{"name": f"item{i}"} for i in range(10)]


@app.get("/items")
async def list_items(
    skip: int = 0,
    limit: Annotated[int, Query(ge=1, le=100)] = 20,
    q: Annotated[str | None, Query(max_length=50)] = None,
    tags: Annotated[list[str], Query()] = [],     # ?tags=a&tags=b
):
    results = FAKE
    if q:
        results = [i for i in results if q.lower() in i["name"]]
    return {"skip": skip, "limit": limit, "q": q, "tags": tags,
            "items": results[skip : skip + limit]}
```

```
GET /items?skip=0&limit=5&q=item&tags=a&tags=b
```

| Pattern | Result |
|---|---|
| `q: str` | Required query parameter |
| `q: str = "x"` | Optional, default `"x"` |
| `q: str \| None = None` | Optional, default `None` |
| `q: list[str] = Query([])` | Repeated key becomes a list |
| `q: Annotated[list[str] \| None, Query()] = None` | Optional repeated key |
| `q: bool = False` | Accepts `true/false/1/0/yes/no/on/off` |

**Alias for awkward names:**

```python
@app.get("/search")
async def search(
    term: Annotated[str, Query(alias="q")] = ...,      # ?q=… maps to `term`
):
    return {"term": term}
```

This also lets you call a field `model_` or `from` — names that would otherwise clash with Python keywords — while exposing a clean query name.

## 7. Request Bodies with Pydantic

```python
from typing import Annotated
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()


class Image(BaseModel):
    url: str
    alt: str | None = None


class Item(BaseModel):
    name: Annotated[str, Field(min_length=1, max_length=100)]
    description: str | None = None
    price: Annotated[float, Field(gt=0)]
    tax: float | None = None
    tags: set[str] = set()
    image: Image | None = None            # nested model
    gallery: list[Image] = []             # list of nested models


@app.post("/items")
async def create_item(item: Item):
    total = item.price + (item.tax or 0)
    return {"item": item, "total": total}


# Body + path + query in one endpoint
@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, important: bool = False):
    return {"item_id": item_id, "important": important, "item": item}


# Multiple body parameters → each becomes a top-level JSON key
class User(BaseModel):
    username: str
    full_name: str | None = None


@app.post("/mixed")
async def mixed(item: Item, user: User):
    # Expects {"item": {...}, "user": {...}}
    return {"item": item, "user": user}


# A single bare body value
from fastapi import Body

@app.post("/echo")
async def echo(payload: Annotated[str, Body(embed=True)]):
    # embed=True nests it: {"payload": "..."}
    return {"received": payload}
```

**Rule:** if a parameter is a Pydantic model, dict, or list, FastAPI reads it from the **body**. If it is a scalar, it comes from the **path** (if named there) or the **query**. Use `Body(embed=True)` when you want a single scalar wrapped in a JSON object.

### Validation error shape

A 422 response body looks like:

```json
{
  "detail": [
    {
      "type": "greater_than",
      "loc": ["body", "price"],
      "msg": "Input should be greater than 0",
      "input": -5,
      "ctx": { "gt": 0 }
    }
  ]
}
```

`loc` is a path into the request — `["body", "price"]`, `["query", "limit"]`, or `["path", "item_id"]`.

## 8. Pydantic v2 Essentials

```python
from datetime import datetime, timezone
from decimal import Decimal
from typing import Annotated, Literal
from pydantic import (
    AliasChoices, BaseModel, ConfigDict, EmailStr, Field,
    HttpUrl, UUID4, computed_field, field_validator, model_validator,
)


class Product(BaseModel):
    # Populate from ORM objects' attributes (SQLAlchemy rows, etc.)
    model_config = ConfigDict(
        from_attributes=True,
        str_strip_whitespace=True,
        extra="forbid",              # reject unknown fields
        frozen=False,
    )

    id: UUID4
    sku: Annotated[str, Field(pattern=r"^[A-Z]{3}-\d{4}$")]
    name: str
    price: Decimal = Field(gt=0, decimal_places=2)
    currency: Literal["GBP", "USD", "EUR"] = "GBP"
    email: EmailStr | None = None
    website: HttpUrl | None = None
    created_at: datetime = Field(default_factory=lambda: datetime.now(timezone.utc))
    internal_note: str | None = Field(default=None, exclude=True)  # never serialised

    # Accept either name when parsing
    title: str | None = Field(
        default=None,
        validation_alias=AliasChoices("title", "headline"),
    )

    # Field-level validation
    @field_validator("name")
    @classmethod
    def name_not_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("name must not be blank")
        return v

    # Cross-field validation
    @model_validator(mode="after")
    def check_currency_for_price(self) -> "Product":
        if self.price > 10_000 and self.currency != "GBP":
            raise ValueError("prices over 10,000 must be in GBP")
        return self

    # Derived field, included in serialisation
    @computed_field
    @property
    def display(self) -> str:
        return f"{self.name} ({self.currency} {self.price})"
```

| Pydantic v2 method | Replaces (v1) |
|---|---|
| `model_dump()` | `.dict()` |
| `model_dump_json()` | `.json()` |
| `model_validate(obj)` | `.parse_obj()` |
| `model_validate_json(s)` | `.parse_raw()` |
| `model_config = ConfigDict(...)` | `class Config:` |
| `ConfigDict(from_attributes=True)` | `ConfigDict(orm_mode=True)` |
| `@field_validator` | `@validator` |
| `@model_validator` | `@root_validator` |
| `Field(pattern=...)` | `Field(regex=...)` |
| `Field(exclude=True)` | `Field(exclude=True)` (unchanged) |

**Type coercion in JSON mode is strict-ish but pragmatic:** `"5"` is accepted for an `int` field, but `"abc"` is rejected. Use `model_config = ConfigDict(strict=True)` to refuse coercion entirely.

## 9. Response Models

`response_model` filters and documents the response — the single most useful safety net, because it prevents accidentally leaking fields like password hashes.

```python
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()


class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr


class UserOut(BaseModel):
    username: str
    email: EmailStr          # no password field at all


@app.post("/users", response_model=UserOut, status_code=201)
async def create_user(user: UserIn) -> UserOut:
    # `password` cannot leak: the response model strips it
    return save(user)


# Excluding unset or None fields
@app.get("/users/{uid}", response_model=UserOut, response_model_exclude_none=True)
async def get_user(uid: int): ...


@app.get("/partial", response_model=UserOut, response_model_exclude_unset=True)
async def partial(): ...
```

| Parameter | Effect |
|---|---|
| `response_model=X` | Serialise and validate as `X` |
| `response_model_exclude_unset=True` | Omit fields never explicitly set |
| `response_model_exclude_none=True` | Omit `None` values |
| `response_model_include={...}` | Whitelist fields |
| `response_model_exclude={...}` | Blacklist fields |
| `response_model_by_alias=False` | Serialise by field name, not alias |

**Return-type annotation works too** (since 0.89) and keeps the signature honest:

```python
@app.post("/users", status_code=201)
async def create_user(user: UserIn) -> UserOut: ...
```

The two are equivalent for filtering: the return annotation **is** used as the response model, so `-> UserOut` strips and validates exactly like `response_model=UserOut`. (It is ignored only when the annotation is not a Pydantic model — e.g. `-> None`.) Prefer whichever keeps the signature honest; `response_model=` wins when you want to keep the narrower schema out of the function's typing, such as returning a wider internal model.

## 10. Response Classes, Headers & Cookies

```python
from fastapi import FastAPI, Response, status
from fastapi.responses import (
    HTMLResponse, JSONResponse, PlainTextResponse, RedirectResponse,
    StreamingResponse, FileResponse, ORJSONResponse,
)

app = FastAPI()


@app.get("/json")
async def json_response():
    return JSONResponse(
        status_code=200,
        content={"ok": True},
        headers={"X-Custom": "value"},
    )


@app.get("/text", response_class=PlainTextResponse)
async def text():
    return "plain text body"


@app.get("/html", response_class=HTMLResponse)
async def html():
    return "<h1>Hello</h1>"


@app.get("/redirect")
async def redirect():
    return RedirectResponse("/text", status_code=status.HTTP_303_SEE_OTHER)


@app.get("/download")
async def download():
    return FileResponse("report.pdf", filename="report.pdf",
                        media_type="application/pdf")


# Streaming (large files, generators, SSE)
@app.get("/stream")
async def stream():
    async def gen():
        for i in range(5):
            yield f"chunk {i}\n"
    return StreamingResponse(gen(), media_type="text/plain")


# Injecting the Response object lets you set headers/cookies AND return a model
@app.post("/login")
async def login(response: Response):
    response.set_cookie(
        "session", "abc123",
        httponly=True, secure=True, samesite="lax",
        max_age=3600, path="/",
    )
    response.headers["X-Token-Issued"] = "true"
    response.delete_cookie("old_cookie")
    return {"logged_in": True}
```

| Response class | Media type |
|---|---|
| `JSONResponse` | `application/json` |
| `ORJSONResponse` | Faster JSON (needs `orjson`) |
| `UJSONResponse` | Faster JSON (needs `ujson`) |
| `PlainTextResponse` | `text/plain` |
| `HTMLResponse` | `text/html` |
| `RedirectResponse` | Redirect with a status code |
| `StreamingResponse` | Any iterable, sync or async |
| `FileResponse` | Async file send with headers |
| `Response` | Bare, for 204-style empty bodies |

**Decorator vs injection:** `response_class=` sets the default class but you can still return a model. Injecting `Response` gives you a handle for headers and cookies while the return value is serialised normally. Use `response_model` filtering with injection carefully — the injected `Response` is not itself filtered.

## 11. Status Codes

```python
from fastapi import FastAPI, status

app = FastAPI()


@app.post("/items", status_code=status.HTTP_201_CREATED)
async def create(): ...


@app.delete("/items/{id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete(id: int):
    return None            # 204 must have no body
```

Return `None` with a 204 — any body is a protocol violation.

| Code | Constant | Meaning |
|---|---|---|
| 200 | `HTTP_200_OK` | Success |
| 201 | `HTTP_201_CREATED` | Resource created |
| 202 | `HTTP_202_ACCEPTED` | Accepted, processing later |
| 204 | `HTTP_204_NO_CONTENT` | Success with no body |
| 301/302 | `HTTP_301_MOVED_PERMANENTLY` | Redirect |
| 304 | `HTTP_304_NOT_MODIFIED` | Conditional request unchanged |
| 307/308 | `HTTP_307_TEMPORARY_REDIRECT` | Method-preserving redirect |
| 400 | `HTTP_400_BAD_REQUEST` | Malformed request |
| 401 | `HTTP_401_UNAUTHORIZED` | Missing or invalid credentials |
| 403 | `HTTP_403_FORBIDDEN` | Authenticated but not permitted |
| 404 | `HTTP_404_NOT_FOUND` | Resource absent |
| 405 | `HTTP_405_METHOD_NOT_ALLOWED` | Wrong HTTP method |
| 409 | `HTTP_409_CONFLICT` | State conflict (duplicate) |
| 422 | `HTTP_422_UNPROCESSABLE_ENTITY` | Validation failed |
| 429 | `HTTP_429_TOO_MANY_REQUESTS` | Rate limited |
| 500 | `HTTP_500_INTERNAL_SERVER_ERROR` | Unhandled server error |
| 503 | `HTTP_503_SERVICE_UNAVAILABLE` | Temporarily down |

Document non-default responses so clients know what to expect:

```python
@app.get(
    "/items/{item_id}",
    responses={
        404: {"description": "Item not found"},
        403: {"description": "Not enough permissions"},
    },
)
async def get_item(item_id: int): ...
```

## 12. Forms, Files & Uploads

Requires `python-multipart` — a missing install produces a startup error, not a request error.

```python
from typing import Annotated
from fastapi import FastAPI, File, Form, UploadFile

app = FastAPI()


# Form fields
@app.post("/login")
async def login(
    username: Annotated[str, Form()],
    password: Annotated[str, Form(min_length=8)],
):
    return {"username": username}


# File upload
@app.post("/upload")
async def upload(file: Annotated[UploadFile, File()]):
    contents = await file.read()          # whole file into memory
    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "size": len(contents),
    }


# Form + file together
@app.post("/profile")
async def profile(
    name: Annotated[str, Form()],
    avatar: Annotated[UploadFile | None, File()] = None,
):
    return {"name": name, "has_avatar": avatar is not None}


# Multiple files
@app.post("/gallery")
async def gallery(files: Annotated[list[UploadFile], File()]):
    return {"count": len(files), "names": [f.filename for f in files]}
```

| `UploadFile` attribute/method | Description |
|---|---|
| `.filename` | Client-supplied filename (untrusted) |
| `.content_type` | Client-supplied MIME type (untrusted) |
| `.file` | The underlying `SpooledTemporaryFile` |
| `await .read(size)` | Read bytes (all, or `size`) |
| `await .seek(0)` | Rewind |
| `await .write(data)` | Write |
| `await .close()` | Close |

**Stream large uploads** rather than `await file.read()`, which loads everything into memory:

```python
import shutil

@app.post("/upload-stream")
async def upload_stream(file: Annotated[UploadFile, File()]):
    dest = f"/tmp/uploads/{file.filename}"
    with open(dest, "wb") as out:
        shutil.copyfileobj(file.file, out)      # chunked copy
    return {"saved": dest, "size": file.size}
```

Never trust `.filename` — it can contain `../` path traversal. Use `pathlib.Path(file.filename).name` and generate your own storage name. Also enforce a size limit and an extension/MIME allowlist.

## 13. Headers & Cookies (Input)

```python
from typing import Annotated
from fastapi import Cookie, Header, FastAPI

app = FastAPI()


@app.get("/headers")
async def read_headers(
    user_agent: Annotated[str | None, Header()] = None,
    x_token: Annotated[list[str] | None, Header()] = None,      # repeated header
    x_request_id: Annotated[str | None, Header(convert_underscores=False)] = None,
):
    # Header names convert underscores to hyphens by default
    return {"user_agent": user_agent, "x_token": x_token}


@app.get("/cookies")
async def read_cookies(
    session_id: Annotated[str | None, Cookie()] = None,
):
    return {"session_id": session_id}
```

`user_agent` maps to `User-Agent`; `x_token` to `X-Token`. Headers are case-insensitive on the wire.

## 14. Dependency Injection

The single most important FastAPI concept after routing. A dependency is any callable that FastAPI runs before your handler and injects the result of.

```python
from typing import Annotated
from fastapi import Depends, FastAPI, Header, HTTPException, status

app = FastAPI()


# Simple dependency
async def common_params(skip: int = 0, limit: int = 20):
    return {"skip": skip, "limit": limit}


@app.get("/items")
async def list_items(commons: Annotated[dict, Depends(common_params)]):
    return commons


# Parameterised dependency (a dependency factory)
def require_role(role: str):
    async def checker(x_user_role: Annotated[str, Header()]) -> str:
        if x_user_role != role:
            raise HTTPException(status.HTTP_403_FORBIDDEN, "Insufficient role")
        return x_user_role
    return checker


@app.get("/admin", dependencies=[Depends(require_role("admin"))])
async def admin_only():
    return {"ok": True}


# Sub-dependencies: dependencies can depend on dependencies
async def get_token(authorization: Annotated[str, Header()]) -> str:
    scheme, _, token = authorization.partition(" ")
    if scheme.lower() != "bearer":
        raise HTTPException(401, "Invalid auth scheme")
    return token


async def get_current_user(
    token: Annotated[str, Depends(get_token)],
) -> dict:
    return decode_and_verify(token)


# Class-based dependency
class Pagination:
    def __init__(self, skip: int = 0, limit: int = 20):
        self.skip = skip
        self.limit = limit


@app.get("/paginated")
async def paginated(p: Annotated[Pagination, Depends(Pagination)]):
    return {"skip": p.skip, "limit": p.limit}
```

### Yield dependencies (resource lifecycle)

```python
from collections.abc import AsyncGenerator, Generator
from fastapi import Depends


def get_db_sync() -> Generator[Session, None, None]:
    db = SessionLocal()
    try:
        yield db                 # given to the handler
    finally:
        db.close()               # always runs, even on exceptions


async def get_db() -> AsyncGenerator[Session, None]:
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
```

**Ordering guarantee:** code before `yield` runs *before* the handler; code after `yield` runs *after* the response. Dependencies are torn down in **reverse order** of setup, and `finally` blocks run even if the handler raised.

### Caching and reuse

Dependencies are **cached per request** — the same dependency called twice in one request resolves once. Disable with `use_cache=False`:

```python
@app.get("/x")
async def x(
    a: Annotated[int, Depends(expensive, use_cache=False)],
    b: Annotated[int, Depends(expensive, use_cache=False)],   # runs twice
): ...


# Global dependencies on the whole app
app = FastAPI(dependencies=[Depends(verify_api_key)])


# On a router
router = APIRouter(dependencies=[Depends(verify_api_key)])
```

| Pattern | Use for |
|---|---|
| Plain function | Shared query params, computed values |
| Factory (`def f(cfg)`) | Parameterised behaviour |
| Class | Stateful config with `__init__` params |
| `yield` generator | DB sessions, file handles, temp state |
| `dependencies=[...]` | Side-effect-only guards (auth, rate limit) |
| `Depends()` in `Depends()` | Composing reusable logic |

**Prefer `Annotated[T, Depends(f)]`** over `x: T = Depends(f)`. The `Annotated` form is reusable as a type alias and avoids default-value ambiguity:

```python
CurrentUser = Annotated[User, Depends(get_current_user)]
DBSession = Annotated[AsyncSession, Depends(get_db)]

@app.get("/me")
async def me(user: CurrentUser, db: DBSession) -> UserOut: ...
```

## 15. Security & Authentication

### Password hashing

```python
from pwdlib import PasswordHash

password_hash = PasswordHash.recommended()      # argon2 by default


def hash_password(plain: str) -> str:
    return password_hash.hash(plain)


def verify_password(plain: str, hashed: str) -> bool:
    return password_hash.verify(plain, hashed)
```

> **Note:** `passlib` + `bcrypt` is a common legacy pairing that breaks with bcrypt ≥ 4.1 (`AttributeError: module 'bcrypt' has no attribute '__about__'`). If you must stay on passlib, pin `bcrypt<4.1`. `pwdlib` avoids the problem entirely and is what current FastAPI docs use.

### Full JWT auth example

```python
# security.py
from datetime import datetime, timedelta, timezone
from typing import Annotated, Any

import jwt
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from jwt.exceptions import InvalidTokenError
from pwdlib import PasswordHash

SECRET_KEY = "change-me-load-from-env"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

password_hash = PasswordHash.recommended()
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


def create_access_token(subject: str, extra: dict[str, Any] | None = None) -> str:
    now = datetime.now(timezone.utc)
    payload = {
        "sub": subject,
        "iat": now,
        "exp": now + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES),
        **(extra or {}),
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)


async def get_current_user(
    token: Annotated[str, Depends(oauth2_scheme)],
) -> "User":
    credentials_error = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    except InvalidTokenError:
        raise credentials_error

    username = payload.get("sub")
    if not username:
        raise credentials_error

    user = await get_user_by_username(username)
    if user is None:
        raise credentials_error
    return user


async def get_current_active_user(
    user: Annotated["User", Depends(get_current_user)],
) -> "User":
    if not user.is_active:
        raise HTTPException(status.HTTP_400_BAD_REQUEST, "Inactive user")
    return user
```

```python
# routes
from typing import Annotated
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm

router = APIRouter()


@router.post("/token")
async def login_for_access_token(
    form: Annotated[OAuth2PasswordRequestForm, Depends()],
):
    user = await authenticate_user(form.username, form.password)
    if not user:
        raise HTTPException(
            status.HTTP_401_UNAUTHORIZED,
            "Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    token = create_access_token(user.username)
    return {"access_token": token, "token_type": "bearer"}


@router.get("/users/me")
async def read_me(user: Annotated["User", Depends(get_current_active_user)]):
    return user
```

`OAuth2PasswordRequestForm` reads **form-encoded** data (`username`, `password`), not JSON — that is the OAuth2 spec, and it is why `/token` needs `python-multipart`. The `tokenUrl="token"` in `OAuth2PasswordBearer` is what makes Swagger UI's **Authorize** button work.

| Security scheme | Reads from |
|---|---|
| `OAuth2PasswordBearer(tokenUrl=...)` | `Authorization: Bearer <token>` |
| `HTTPBearer()` | `Authorization: Bearer <token>`, returns an object |
| `HTTPBasic()` | `Authorization: Basic <base64>` |
| `APIKeyHeader(name="X-API-Key")` | A header |
| `APIKeyQuery(name="api_key")` | A query parameter |
| `APIKeyCookie(name="session")` | A cookie |
| `OAuth2AuthorizationCodeBearer` | Full OAuth2 code flow |

### Scopes

```python
from fastapi.security import SecurityScopes

oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="token",
    scopes={"items:read": "Read items", "items:write": "Modify items"},
)


async def get_user_with_scopes(
    security_scopes: SecurityScopes,
    token: Annotated[str, Depends(oauth2_scheme)],
):
    payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    token_scopes = set(payload.get("scopes", []))
    for scope in security_scopes.scopes:
        if scope not in token_scopes:
            raise HTTPException(
                status.HTTP_403_FORBIDDEN,
                f"Missing scope: {scope}",
                headers={"WWW-Authenticate": f'Bearer scope="{security_scopes.scope_str}"'},
            )
    return payload


@app.get("/items/write")
async def write_item(
    user: Annotated[dict, Security(get_user_with_scopes, scopes=["items:write"])],
): ...
```

**Auth rules that matter:** always load `SECRET_KEY` from the environment, never commit it. Set `exp` on every token and validate `algorithms=[...]` explicitly — passing a list prevents the `alg: none` and algorithm-confusion attacks. Store passwords with argon2 or bcrypt, never SHA-256 alone.

## 16. Middleware & CORS

```python
import time
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.gzip import GZipMiddleware
from starlette.middleware.trustedhost import TrustedHostMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://app.example.com"],   # never "*" with credentials
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "PATCH", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
    expose_headers=["X-Request-ID"],
    max_age=600,
)

app.add_middleware(GZipMiddleware, minimum_size=1000)
app.add_middleware(TrustedHostMiddleware, allowed_hosts=["example.com", "*.example.com"])


# Custom middleware via decorator
@app.middleware("http")
async def add_timing_header(request: Request, call_next):
    start = time.perf_counter()
    response = await call_next(request)
    elapsed = (time.perf_counter() - start) * 1000
    response.headers["X-Process-Time-Ms"] = f"{elapsed:.1f}"
    return response


# Middleware as a class (reusable, composable)
import uuid

from starlette.middleware.base import BaseHTTPMiddleware


class RequestIDMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        request_id = request.headers.get("X-Request-ID", str(uuid.uuid4()))
        request.state.request_id = request_id
        response = await call_next(request)
        response.headers["X-Request-ID"] = request_id
        return response


app.add_middleware(RequestIDMiddleware)
```

**Middleware ordering:** middleware added **last** runs **first** (outermost). Think of it as wrapping layers.

**CORS trap:** `allow_origins=["*"]` is incompatible with `allow_credentials=True` — the browser rejects the wildcard when credentials are in play. Always list explicit origins if you need cookies or auth headers.

**Prefer pure ASGI middleware over `BaseHTTPMiddleware`** for performance-sensitive paths: `BaseHTTPMiddleware` wraps every request in an extra task, which adds measurable overhead and can interfere with streaming and `BackgroundTasks`.

## 17. Routers & Project Structure

```python
# app/routers/items.py
from typing import Annotated
from fastapi import APIRouter, Depends, HTTPException, status

from app.deps import CurrentUser, DBSession
from app.schemas import ItemCreate, ItemOut, ItemUpdate

router = APIRouter(
    prefix="/items",
    tags=["items"],
    dependencies=[Depends(get_current_user)],       # applies to every route
    responses={404: {"description": "Not found"}},
)


@router.get("", response_model=list[ItemOut])
async def list_items(db: DBSession, user: CurrentUser, skip: int = 0, limit: int = 20):
    ...


@router.get("/{item_id}", response_model=ItemOut)
async def get_item(item_id: int, db: DBSession, user: CurrentUser):
    item = await db.get(Item, item_id)
    if item is None:
        raise HTTPException(status.HTTP_404_NOT_FOUND, "Item not found")
    return item


@router.post("", response_model=ItemOut, status_code=status.HTTP_201_CREATED)
async def create_item(payload: ItemCreate, db: DBSession, user: CurrentUser):
    ...
```

```python
# app/main.py
from fastapi import FastAPI

from app.routers import items, users

app = FastAPI(title="My API", version="1.0.0")

app.include_router(items.router)
app.include_router(users.router, prefix="/api/v1")       # extra prefix
```

```python
# app/routers/__init__.py
from app.routers import items, users

__all__ = ["items", "users"]
```

### Recommended layout (grows with the app)

```
myapp/
├── app/
│   ├── __init__.py
│   ├── main.py            # FastAPI app, routers, middleware, lifespan
│   ├── config.py          # settings via pydantic-settings
│   ├── database.py        # engine, session factory, Base
│   ├── models.py          # SQLAlchemy ORM models
│   ├── schemas.py         # Pydantic request/response models
│   ├── security.py        # hashing, JWT, auth dependencies
│   ├── deps.py            # shared Annotated dependency aliases
│   ├── crud.py            # data-access helpers
│   └── routers/
│       ├── __init__.py
│       ├── items.py
│       └── users.py
├── tests/
│   ├── conftest.py
│   └── test_items.py
├── alembic/               # migrations
├── .env                   # NEVER committed
├── pyproject.toml
└── Dockerfile
```

**Key conventions:** keep ORM models (`models.py`) and API schemas (`schemas.py`) separate — never return a SQLAlchemy model directly from an endpoint. Define shared dependency aliases once in `deps.py` and reuse them, so signatures stay short and consistent.

## 18. Error Handling

```python
from fastapi import FastAPI, HTTPException, Request, status
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()


# Raising errors in handlers
@app.get("/items/{item_id}")
async def get_item(item_id: int):
    if item_id > 100:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Item not found",
            headers={"X-Error": "not-found"},
        )
    return {"item_id": item_id}


# Custom exception class
class DomainError(Exception):
    def __init__(self, code: str, message: str):
        self.code = code
        self.message = message


# Register a handler
@app.exception_handler(DomainError)
async def domain_error_handler(request: Request, exc: DomainError):
    return JSONResponse(
        status_code=status.HTTP_409_CONFLICT,
        content={"error": {"code": exc.code, "message": exc.message}},
    )


# Override the default 404 shape
@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request: Request, exc: StarletteHTTPException):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": {
                "status": exc.status_code,
                "detail": exc.detail,
                "path": request.url.path,
            }
        },
    )


# Consistent validation errors
@app.exception_handler(RequestValidationError)
async def validation_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={
            "error": {
                "code": "validation_error",
                "fields": [
                    {"loc": e["loc"], "msg": e["msg"], "type": e["type"]}
                    for e in exc.errors()
                ],
            }
        },
    )


# Catch-all (log it; never leak internals to the client)
import logging

logger = logging.getLogger(__name__)


@app.exception_handler(Exception)
async def unhandled_handler(request: Request, exc: Exception):
    logger.exception("Unhandled error on %s", request.url.path)
    return JSONResponse(status_code=500, content={"error": {"code": "internal_error"}})
```

Note: `@app.exception_handler(Exception)` only catches errors raised in the ASGI stack; errors in middleware added *outside* the handler may bypass it. Registering it on the app is the usual approach, and it must be the last resort — do not use it to swallow bugs silently.

| Approach | Use for |
|---|---|
| `raise HTTPException(...)` | Expected client-facing failures |
| Custom exception + handler | Domain errors with consistent shapes |
| `RequestValidationError` handler | Normalising 422 responses |
| `Exception` handler | Logging and a safe generic 500 body |
| `JSONResponse` return | Non-exception paths needing a custom body |

## 19. Background Tasks

```python
from fastapi import BackgroundTasks, FastAPI, status

app = FastAPI()


def send_email(to: str, subject: str) -> None:
    ...


async def write_audit(entry: dict) -> None:
    ...


@app.post("/signup", status_code=status.HTTP_202_ACCEPTED)
async def signup(email: str, background: BackgroundTasks):
    background.add_task(send_email, email, "Welcome")
    background.add_task(write_audit, {"event": "signup", "email": email})
    return {"queued": True}
```

**What `BackgroundTasks` is and is not:**

| It is | It is not |
|---|---|
| Code that runs after the response is sent | A durable job queue |
| In-process, same worker | Survivable across restarts |
| Fine for emails, audit logs, cache warmup | Suitable for long or retryable work |
| Simple to use | Retried on failure |

For anything that must survive a crash, be retried, or be deduplicated, use a real broker — Celery, RQ, Dramatiq, or Arq — and push a task from the endpoint instead.

## 20. Lifespan & Application State

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI, Request

from app.database import engine
from app.redis import redis_client


@asynccontextmanager
async def lifespan(app: FastAPI):
    # --- startup ---
    app.state.http = create_http_client()
    app.state.redis = await redis_client()
    await warm_cache()
    yield
    # --- shutdown ---
    await app.state.redis.aclose()
    await app.state.http.aclose()
    await engine.dispose()


app = FastAPI(lifespan=lifespan)


@app.get("/cached/{key}")
async def cached(key: str, request: Request):
    # Reach shared resources through app.state
    value = await request.app.state.redis.get(key)
    return {"value": value}
```

`@app.on_event("startup")` / `("shutdown")` still works but is deprecated — lifespan is the supported replacement. Caveat: with the `asynccontextmanager` form, if startup raises before the `yield`, the code after `yield` never runs — the app just fails to start. Teardown is guaranteed only once the `yield` was reached; wrap startup work in `try`/`finally` inside the generator if partial-failure cleanup matters.

**Worker reality check:** with `--workers 4` you get **four independent processes**, each running the lifespan handler and each with its own `app.state`. Anything shared must live outside the process — Redis, Postgres, a message bus. In-process caches will not be coherent across workers.

## 21. Async vs Sync

```python
import httpx
from fastapi import FastAPI

app = FastAPI()


# async def → runs on the event loop. Use for I/O with async libraries.
@app.get("/async")
async def async_handler():
    async with httpx.AsyncClient() as client:
        r = await client.get("https://api.example.com/data")
    return r.json()


# def → runs in a threadpool. Use for blocking I/O or sync-only libraries.
@app.get("/sync")
def sync_handler():
    import requests
    return requests.get("https://api.example.com/data").json()
```

| Handler | Runs on | Use when |
|---|---|---|
| `async def` | The event loop | Every await is non-blocking |
| `def` | A threadpool (default 40 threads) | Any blocking I/O or CPU-light sync code |

**The cardinal rule:** never call blocking code inside `async def`. A single `requests.get()` or `time.sleep()` in an async handler blocks the **entire event loop** — all other requests stall until it returns.

```python
# WRONG — blocks the whole server
@app.get("/bad")
async def bad():
    import requests
    return requests.get("https://slow.example.com").json()   # freezes everything

# RIGHT — either await an async client…
@app.get("/good1")
async def good1():
    async with httpx.AsyncClient() as client:
        return (await client.get("https://slow.example.com")).json()

# …or declare it sync so FastAPI offloads it to a thread
@app.get("/good2")
def good2():
    import requests
    return requests.get("https://slow.example.com").json()

# …or offload explicitly
import anyio.to_thread

@app.get("/good3")
async def good3():
    return await anyio.to_thread.run_sync(blocking_call)
```

| Situation | Recommendation |
|---|---|
| HTTP calls | `httpx.AsyncClient` in `async def` |
| Database | Async driver (`asyncpg`, `aiomysql`) + async SQLAlchemy |
| Redis | `redis.asyncio` |
| CPU-heavy work | A separate process or task queue — not the API worker |
| Sync-only library | `def` handler, or `anyio.to_thread.run_sync` |
| File I/O | `anyio.Path` or a threadpool |

**Do not** run heavy CPU work (image resizing, ML inference, large sorts) in the API process — it starves every other request on that worker. Push it to a worker pool or a task queue.

## 22. Database Integration

### Async SQLAlchemy 2.0

```python
# app/database.py
from collections.abc import AsyncGenerator

from sqlalchemy.ext.asyncio import (
    AsyncSession, async_sessionmaker, create_async_engine,
)
from sqlalchemy.orm import DeclarativeBase

DATABASE_URL = "postgresql+asyncpg://user:pass@localhost:5432/mydb"

engine = create_async_engine(
    DATABASE_URL,
    echo=False,
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True,       # detect dead connections
)

AsyncSessionLocal = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,    # keep attributes usable after commit
    autoflush=False,
)


class Base(DeclarativeBase):
    pass


async def get_session() -> AsyncGenerator[AsyncSession, None]:
    async with AsyncSessionLocal() as session:
        try:
            yield session
        except Exception:
            await session.rollback()
            raise
```

```python
# app/models.py
from datetime import datetime
from sqlalchemy import ForeignKey, String, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.database import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    username: Mapped[str] = mapped_column(String(50), unique=True, index=True)
    email: Mapped[str] = mapped_column(String(255), unique=True)
    hashed_password: Mapped[str] = mapped_column(String(255))
    is_active: Mapped[bool] = mapped_column(default=True)
    created_at: Mapped[datetime] = mapped_column(server_default=func.now())

    items: Mapped[list["Item"]] = relationship(
        back_populates="owner", cascade="all, delete-orphan",
    )


class Item(Base):
    __tablename__ = "items"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(100), index=True)
    price: Mapped[float]
    owner_id: Mapped[int] = mapped_column(ForeignKey("users.id"))

    owner: Mapped["User"] = relationship(back_populates="items")
```

```python
# app/crud.py
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.models import Item


async def get_item(db: AsyncSession, item_id: int) -> Item | None:
    return await db.get(Item, item_id)


async def list_items(db: AsyncSession, skip: int = 0, limit: int = 20) -> list[Item]:
    result = await db.execute(
        select(Item).order_by(Item.id).offset(skip).limit(limit)
    )
    return list(result.scalars().all())


async def create_item(db: AsyncSession, **fields) -> Item:
    item = Item(**fields)
    db.add(item)
    await db.commit()
    await db.refresh(item)
    return item
```

| Pitfall | Consequence |
|---|---|
| Sync engine with `async def` handlers | Blocks the event loop |
| Missing `await session.commit()` | Data silently not persisted |
| `lazy="select"` relationships with async | `MissingGreenlet` error on attribute access |
| Sharing one session across requests | Cross-request state bleed; use the dependency |
| No `pool_pre_ping` | Stale connections after a DB restart |
| No unique-constraint handling | `IntegrityError` surfaces as a 500 |
| N+1 queries | One query per row; use `selectinload` |

Use `selectinload`/`joinedload` for relationships in async code, because lazy loading cannot run outside the greenlet context:

```python
from sqlalchemy.orm import selectinload

result = await db.execute(
    select(User).options(selectinload(User.items)).where(User.id == user_id)
)
```

### Migrations (Alembic)

```bash
alembic init -t async alembic          # async template
# edit alembic/env.py: set target_metadata = Base.metadata and the URL
alembic revision --autogenerate -m "add items table"
alembic upgrade head
alembic downgrade -1
alembic current                        # which revision is applied
alembic history --verbose
```

Always review autogenerated migrations — Alembic infers from model diffs and misses renames, data migrations, and some constraint changes.

**Also worth knowing:** `SQLModel` merges ORM and API schemas into one class (a FastAPI author's project) — convenient for small apps, but it couples your wire format to your database schema, which hurts as the app grows.

## 23. Settings & Configuration

```python
# app/config.py
from functools import lru_cache
from pydantic import Field, PostgresDsn, field_validator
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        extra="ignore",
        case_sensitive=False,
    )

    app_name: str = "My API"
    environment: str = "development"
    debug: bool = False

    secret_key: str
    access_token_expire_minutes: int = 30

    database_url: PostgresDsn
    redis_url: str | None = None

    cors_origins: list[str] = Field(default_factory=list)

    @field_validator("cors_origins", mode="before")
    @classmethod
    def split_origins(cls, v):
        # Allow CORS_ORIGINS="a,b,c" in the environment
        if isinstance(v, str) and not v.startswith("["):
            return [o.strip() for o in v.split(",") if o.strip()]
        return v

    @field_validator("secret_key")
    @classmethod
    def no_default_secret(cls, v: str) -> str:
        if len(v) < 32:
            raise ValueError("secret_key must be at least 32 characters")
        return v


@lru_cache
def get_settings() -> Settings:
    return Settings()


settings = get_settings()
```

```bash
# .env  — NEVER commit this file
SECRET_KEY=change-me-to-a-long-random-string-at-least-32-chars
DATABASE_URL=postgresql+asyncpg://user:pass@localhost:5432/mydb
CORS_ORIGINS=https://app.example.com,https://admin.example.com
```

| Rule | Reason |
|---|---|
| `.env` in `.gitignore` | Secrets must never enter git history |
| Commit `.env.example` instead | Documents required variables |
| Validate at startup, not at request time | Fail fast on misconfiguration |
| Use `lru_cache` on the getter | Parse the environment once |
| Inject settings via `Depends(get_settings)` | Makes testing trivial |
| Never log the populated object | It contains the secret key |

Add `Depends(get_settings)` to endpoints that need config rather than importing the module-level singleton — that is what makes overrides possible in tests.

## 24. Templates & Static Files

```python
from pathlib import Path
from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates

app = FastAPI()

app.mount("/static", StaticFiles(directory="static"), name="static")

templates = Jinja2Templates(directory="templates")


@app.get("/page", response_class=HTMLResponse)
async def page(request: Request, name: str = "World"):
    # Current signature: request is the FIRST argument
    return templates.TemplateResponse(
        request, "index.html", {"name": name},
    )
```

```html
<!-- templates/index.html -->
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>{{ title or "My App" }}</title>
    <link rel="stylesheet" href="{{ url_for('static', path='app.css') }}">
  </head>
  <body>
    <h1>Hello {{ name }}</h1>
  </body>
</html>
```

`url_for('static', path=...)` requires the `name="static"` argument on the mount. Jinja2 auto-escapes HTML by default — do not disable it with `|safe` on user data.

## 25. WebSockets & Server-Sent Events

### WebSockets

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect

app = FastAPI()


class ConnectionManager:
    def __init__(self) -> None:
        self.active: list[WebSocket] = []

    async def connect(self, ws: WebSocket) -> None:
        await ws.accept()
        self.active.append(ws)

    def disconnect(self, ws: WebSocket) -> None:
        self.active.remove(ws)

    async def broadcast(self, message: str) -> None:
        for ws in list(self.active):
            await ws.send_text(message)


manager = ConnectionManager()


@app.websocket("/ws/{client_id}")
async def websocket_endpoint(ws: WebSocket, client_id: str):
    await manager.connect(ws)
    try:
        while True:
            data = await ws.receive_text()
            await manager.broadcast(f"[{client_id}] {data}")
    except WebSocketDisconnect:
        manager.disconnect(ws)
```

WebSocket endpoints **do** support `Depends`, `Security`, `Cookie`, `Header`, `Path`, and `Query` — the values come from the HTTP handshake request, and raising `HTTPException` (or `WebSocketException`) in a dependency rejects the connection. So you can authenticate with a dependency like on any route. In practice browsers cannot set headers on the handshake, so tokens usually arrive via query string or a `Sec-WebSocket-Protocol`/cookie header; if you must read the token from the first message instead, do it inside the handler and `await ws.close(code=1008)` on failure.

### Server-Sent Events

```python
import asyncio
import json
from fastapi.responses import StreamingResponse

from fastapi import FastAPI

app = FastAPI()


@app.get("/events")
async def events():
    async def event_stream():
        for i in range(10):
            payload = json.dumps({"tick": i})
            yield f"event: tick\ndata: {payload}\n\n"   # SSE frame format
            await asyncio.sleep(1)

    return StreamingResponse(
        event_stream(),
        media_type="text/event-stream",
        headers={"Cache-Control": "no-cache", "X-Accel-Buffering": "no"},
    )
```

The SSE frame format is mandatory: `event:`, `data:`, and a blank line as the terminator. `X-Accel-Buffering: no` prevents nginx from buffering the stream.

> Recent FastAPI releases added first-class SSE helpers (`fastapi.sse`). Check whether your installed version provides them before hand-rolling the framing.

## 26. Testing

### Sync with `TestClient`

```python
# tests/test_main.py
from fastapi.testclient import TestClient

from app.main import app

client = TestClient(app)


def test_root():
    r = client.get("/")
    assert r.status_code == 200
    assert r.json() == {"message": "Hello World"}


def test_create_item_validates():
    r = client.post("/items", json={"name": "", "price": -1})
    assert r.status_code == 422
    assert any(e["loc"] == ["body", "price"] for e in r.json()["detail"])
```

### Async with `httpx`

```python
# tests/test_async.py
import pytest
from httpx import ASGITransport, AsyncClient

from app.main import app


@pytest.fixture
def anyio_backend():
    return "asyncio"


@pytest.mark.anyio
async def test_list_items():
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as ac:
        r = await ac.get("/items", params={"limit": 5})
    assert r.status_code == 200
    assert len(r.json()["items"]) <= 5
```

### Overriding dependencies

```python
# tests/conftest.py
import anyio
import pytest
from fastapi.testclient import TestClient
from sqlalchemy.ext.asyncio import async_sessionmaker, create_async_engine

from app.database import Base, get_session
from app.main import app

TEST_URL = "postgresql+asyncpg://user:pass@localhost:5432/testdb"


@pytest.fixture
def client():
    # Sync fixture: drive the async engine with anyio.run, as in §32
    engine = create_async_engine(TEST_URL)
    factory = async_sessionmaker(engine, expire_on_commit=False)

    async def setup():
        async with engine.begin() as conn:
            await conn.run_sync(Base.metadata.create_all)

    anyio.run(setup)

    async def override_get_session():
        async with factory() as session:
            yield session

    app.dependency_overrides[get_session] = override_get_session
    with TestClient(app) as c:
        yield c
    app.dependency_overrides.clear()

    async def teardown():
        async with engine.begin() as conn:
            await conn.run_sync(Base.metadata.drop_all)
        await engine.dispose()

    anyio.run(teardown)
```

```python
# tests/test_items.py
def test_create_and_read(client):
    r = client.post("/items", json={"name": "Widget", "price": 9.99})
    assert r.status_code == 201
    item_id = r.json()["id"]

    r = client.get(f"/items/{item_id}")
    assert r.status_code == 200
    assert r.json()["name"] == "Widget"
```

| Test approach | When |
|---|---|
| `TestClient` | Simple, synchronous, easy — the default |
| `httpx.AsyncClient` + `ASGITransport` | Testing async handlers and lifespans |
| `dependency_overrides` | Swap the DB, clock, or auth for fakes |
| `pytest.mark.anyio` | Async test marker (via the `anyio` plugin) |
| Transaction rollback fixture | Fast isolation without recreating tables |
| Real Postgres via testcontainers | Highest fidelity; slower |
| SQLite for tests | **Careful** — behaviour differs from Postgres |

**Always call `app.dependency_overrides.clear()`** after a test. Leaked overrides produce baffling failures in later tests.

## 27. OpenAPI & Docs Customisation

```python
from fastapi import FastAPI
from fastapi.openapi.utils import get_openapi

tags_metadata = [
    {"name": "items", "description": "Create and read **items**."},
    {"name": "users", "description": "User accounts and authentication."},
]

app = FastAPI(
    title="My API",
    version="1.0.0",
    description="An example API.",
    summary="Short one-liner for listings.",
    contact={"name": "API Team", "email": "api@example.com"},
    license_info={"name": "MIT", "url": "https://opensource.org/licenses/MIT"},
    openapi_tags=tags_metadata,
    docs_url="/docs",          # set to None to disable
    redoc_url="/redoc",
    openapi_url="/openapi.json",
    root_path="/api",           # when behind a proxy that strips a prefix
)


def custom_openapi():
    if app.openapi_schema:
        return app.openapi_schema
    schema = get_openapi(
        title=app.title,
        version=app.version,
        description=app.description,
        routes=app.routes,
    )
    schema["info"]["x-logo"] = {"url": "https://example.com/logo.png"}
    schema.setdefault("components", {})["securitySchemes"] = {
        "ApiKeyAuth": {"type": "apiKey", "in": "header", "name": "X-API-Key"}
    }
    app.openapi_schema = schema         # cache it — regenerating is expensive
    return schema


app.openapi = custom_openapi
```

| Item | Purpose |
|---|---|
| `openapi_tags` | Descriptions and ordering in the docs sidebar |
| `OpenAPI(openapi_url=None)` | Disable the schema entirely (production hardening) |
| `/docs` vs `/redoc` | Interactive vs read-only docs |
| `root_path` | Correct URLs when mounted behind a path prefix |
| `servers=[{"url": ...}]` | Override the server list in the schema |
| `responses={...}` | Document error shapes per route |
| `app.openapi_schema` cache | Avoid rebuilding the schema per request |

**Production note:** consider disabling or protecting `/docs`, `/redoc`, and `/openapi.json` in production — they advertise your full attack surface.

## 28. Deployment

### Dockerfile

```dockerfile
FROM python:3.12-slim AS base

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /app

# Install dependencies first so this layer caches
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY ./app ./app

# Never run as root
RUN useradd --create-home --uid 10001 appuser && chown -R appuser /app
USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8000/health')"

CMD ["fastapi", "run", "app/main.py", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

```yaml
# docker-compose.yml
services:
  api:
    build: .
    ports: ["8000:8000"]
    environment:
      DATABASE_URL: postgresql+asyncpg://postgres:postgres@db:5432/app
      SECRET_KEY: ${SECRET_KEY}
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: app
    volumes: ["pgdata:/var/lib/postgresql/data"]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      retries: 5

volumes:
  pgdata:
```

### Production checklist

| Concern | Action |
|---|---|
| Workers | Async workers multiplex connections, so ≈ CPU cores is often plenty for I/O-bound work (`2 × cores + 1` is the sync-worker rule); measure |
| HTTPS | Terminate TLS at a proxy or load balancer, not in the app |
| `root_path` | Set when a proxy strips a path prefix |
| Proxies | Configure `--proxy-headers` and `--forwarded-allow-ips` so client IPs are real |
| Health check | A cheap `/health` that avoids hitting the DB on every probe |
| Graceful shutdown | Lifespan handles the ASGI lifespan protocol; give it time to drain |
| Logging | Structured JSON to stdout; let the platform collect it |
| Migrations | Run `alembic upgrade head` as a separate step, not on app start with multiple workers |
| Secrets | Environment or a secret manager — never baked into the image |
| Rate limiting | At the gateway, or with `slowapi`/middleware |
| Docs | Disable or protect in production |
| Timeouts | Set request timeouts at the proxy; unhandled slow handlers exhaust workers |

**Do not run migrations from the app's startup when you have multiple workers** — every worker races to apply them. Use a discrete init/Job step.

## 29. Performance

| Technique | Impact |
|---|---|
| Async handlers with async libraries | High — avoids event-loop blocking |
| `ORJSONResponse` as the default | Moderate — faster JSON encode |
| `selectinload` instead of lazy loading | High — kills N+1 queries |
| Connection pooling with sane sizes | High — avoids connection storms |
| Response model filtering | Low CPU, plus it prevents data leaks |
| `GZipMiddleware` | Network savings; small CPU cost |
| Pagination on every list endpoint | Prevents unbounded result sets |
| Caching (Redis) for hot reads | High for read-heavy endpoints |
| `GZipMiddleware` before `CORSMiddleware`? | Ordering matters — test it |
| `BackgroundTasks` for side effects | Keeps responses fast |
| Avoid `BaseHTTPMiddleware` on hot paths | Removes per-request task overhead |
| Separate workers for CPU work | Keeps API latency predictable |
| `uvloop` (linux) | Modest event-loop speedup |
| Database indexes on filtered columns | Often the single biggest win |

```python
# Faster JSON globally
from fastapi.responses import ORJSONResponse
app = FastAPI(default_response_class=ORJSONResponse)


# Realistic health check — cheap, no DB hit
@app.get("/health", include_in_schema=False)
async def health():
    return {"status": "ok"}
```

Measure before optimising. `py-spy` and `--log-level debug` with timing middleware will find the real bottleneck far faster than guessing.

## 30. CLI Commands

| Command | Purpose |
|---|---|
| `fastapi dev main.py` | Dev server with reload |
| `fastapi dev --port 8080` | Dev server on another port |
| `fastapi run main.py --workers 4` | Production server |
| `uvicorn main:app --reload` | Equivalent dev server |
| `uvicorn main:app --host 0.0.0.0 --port 8000` | Bind all interfaces |
| `uvicorn main:app --workers 4` | Multiple worker processes |
| `uvicorn main:app --log-level debug` | Verbose logging |
| `uvicorn main:app --proxy-headers --forwarded-allow-ips='*'` | Trust proxy headers |
| `uvicorn main:app --root-path /api` | Path prefix behind a proxy |
| `uvicorn main:app --limit-concurrency 100` | Back-pressure guard |
| `gunicorn main:app -k uvicorn_worker.UvicornWorker -w 4` | Gunicorn-managed workers (`pip install uvicorn-worker`; `uvicorn.workers` is deprecated) |
| `pip install "fastapi[standard]"` | Install core plus the CLI |
| `alembic upgrade head` | Apply migrations |
| `alembic revision --autogenerate -m msg` | Create a migration |
| `pytest -q` | Run tests |
| `pytest --cov=app --cov-report=term-missing` | Tests with coverage |
| `ruff check . --fix` | Lint and autofix |
| `mypy app` | Type-check |
| `docker compose up --build` | Local stack |

## 31. Common Pitfalls

| Pitfall | Symptom | Fix |
|---|---|---|
| Blocking call in `async def` | Whole server stalls | `def` handler or async client |
| Missing `await` | Coroutine never runs; silent no-op | Await every coroutine |
| Sync DB driver in async handler | Event-loop blocking | `asyncpg`/`aiomysql` + async engine |
| Missing `python-multipart` | Startup error on `Form`/`File` | Install it |
| Returning an ORM model | Leaks columns; serialisation errors | Separate schemas + `response_model` |
| `response_model` omitted | Password/hash fields leak | Always set it on write endpoints |
| Lazy-loading relationship | `MissingGreenlet` error | `selectinload` / `joinedload` |
| `allow_origins=["*"]` with credentials | Browser blocks the request | List explicit origins |
| Route declared out of order | `/items/me` shadowed by `/items/{id}` | Literal routes first |
| Shared mutable DB session | Cross-request contamination | One session per request via `Depends` |
| `@app.on_event` | Deprecation warnings | Use `lifespan` |
| Multiple workers, in-process cache | Incoherent cache | External cache (Redis) |
| Migrations in startup | Workers race | Separate deploy step |
| `except Exception` swallowing errors | Silent 200s on failure | Log and re-raise |
| Unbounded `limit` | Query returns everything | `Query(le=100)` |
| Unvalidated `UploadFile.filename` | Path traversal | `Path(name).name` + allowlist |
| `"detail"` assumed in errors | `KeyError` when the shape differs | Custom validation handler |
| `dependency_overrides` not cleared | Flaky tests | `clear()` in teardown |
| `SECRET_KEY` in the repo | Full token forgery | Load from the environment |
| `jwt.decode` without `algorithms=[...]` | Algorithm-confusion attack | Always pass the list |
| No pagination | Memory blow-up under load | Offset/cursor pagination |
| Unclosed clients/resources | Connection leaks | Lifespan teardown, `aclose()` |

## 32. Full Worked Example

A complete, runnable items API with auth, a database, and tests.

### Project layout

```
fastapi-demo/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── database.py
│   ├── models.py
│   ├── schemas.py
│   ├── security.py
│   ├── deps.py
│   └── routers/
│       ├── __init__.py
│       ├── auth.py
│       └── items.py
├── tests/
│   ├── conftest.py
│   └── test_items.py
├── requirements.txt
├── .env
└── .env.example
```

### `app/config.py`

```python
from functools import lru_cache

from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", extra="ignore")

    app_name: str = "Items API"
    debug: bool = False
    secret_key: str
    access_token_expire_minutes: int = 30
    database_url: str = "sqlite+aiosqlite:///./demo.db"
    cors_origins: list[str] = []


@lru_cache
def get_settings() -> Settings:
    return Settings()
```

### `app/database.py`

```python
from collections.abc import AsyncGenerator

from sqlalchemy.ext.asyncio import (
    AsyncSession, async_sessionmaker, create_async_engine,
)
from sqlalchemy.orm import DeclarativeBase

from app.config import get_settings

settings = get_settings()

engine = create_async_engine(settings.database_url, echo=settings.debug)
SessionLocal = async_sessionmaker(engine, expire_on_commit=False)


class Base(DeclarativeBase):
    pass


async def get_session() -> AsyncGenerator[AsyncSession, None]:
    async with SessionLocal() as session:
        try:
            yield session
        except Exception:
            await session.rollback()
            raise
```

### `app/models.py`

```python
from datetime import datetime, timezone

from sqlalchemy import DateTime, ForeignKey, String, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.database import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    username: Mapped[str] = mapped_column(String(50), unique=True, index=True)
    hashed_password: Mapped[str] = mapped_column(String(255))
    is_active: Mapped[bool] = mapped_column(default=True)

    items: Mapped[list["Item"]] = relationship(
        back_populates="owner", cascade="all, delete-orphan",
    )


class Item(Base):
    __tablename__ = "items"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(100), index=True)
    description: Mapped[str | None] = mapped_column(String(500), default=None)
    price: Mapped[float]
    owner_id: Mapped[int] = mapped_column(ForeignKey("users.id"))
    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), server_default=func.now(),
    )

    owner: Mapped["User"] = relationship(back_populates="items")
```

### `app/schemas.py`

```python
from datetime import datetime
from typing import Annotated

from pydantic import BaseModel, ConfigDict, Field


class Token(BaseModel):
    access_token: str
    token_type: str = "bearer"


class UserCreate(BaseModel):
    username: Annotated[str, Field(min_length=3, max_length=50)]
    password: Annotated[str, Field(min_length=8)]


class UserOut(BaseModel):
    model_config = ConfigDict(from_attributes=True)

    id: int
    username: str
    is_active: bool


class ItemBase(BaseModel):
    name: Annotated[str, Field(min_length=1, max_length=100)]
    description: Annotated[str | None, Field(max_length=500)] = None
    price: Annotated[float, Field(gt=0)]


class ItemCreate(ItemBase):
    pass


class ItemUpdate(BaseModel):
    name: str | None = None
    description: str | None = None
    price: Annotated[float | None, Field(gt=0)] = None


class ItemOut(ItemBase):
    model_config = ConfigDict(from_attributes=True)

    id: int
    owner_id: int
    created_at: datetime
```

### `app/security.py`

```python
from datetime import datetime, timedelta, timezone

import jwt
from pwdlib import PasswordHash

from app.config import get_settings

settings = get_settings()
password_hash = PasswordHash.recommended()

ALGORITHM = "HS256"


def hash_password(plain: str) -> str:
    return password_hash.hash(plain)


def verify_password(plain: str, hashed: str) -> bool:
    return password_hash.verify(plain, hashed)


def create_access_token(subject: str) -> str:
    now = datetime.now(timezone.utc)
    payload = {
        "sub": subject,
        "iat": now,
        "exp": now + timedelta(minutes=settings.access_token_expire_minutes),
    }
    return jwt.encode(payload, settings.secret_key, algorithm=ALGORITHM)


def decode_token(token: str) -> dict:
    return jwt.decode(token, settings.secret_key, algorithms=[ALGORITHM])
```

### `app/deps.py`

```python
from typing import Annotated

import jwt
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.database import get_session
from app.models import User
from app.security import decode_token

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/token")

DBSession = Annotated[AsyncSession, Depends(get_session)]


async def get_current_user(
    token: Annotated[str, Depends(oauth2_scheme)],
    db: DBSession,
) -> User:
    creds_error = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = decode_token(token)
    except jwt.InvalidTokenError:
        raise creds_error

    username = payload.get("sub")
    if not username:
        raise creds_error

    result = await db.execute(select(User).where(User.username == username))
    user = result.scalar_one_or_none()
    if user is None:
        raise creds_error
    return user


CurrentUser = Annotated[User, Depends(get_current_user)]


async def get_active_user(user: CurrentUser) -> User:
    if not user.is_active:
        raise HTTPException(status.HTTP_403_FORBIDDEN, "Inactive user")
    return user


ActiveUser = Annotated[User, Depends(get_active_user)]
```

### `app/routers/auth.py`

```python
from typing import Annotated

from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm
from sqlalchemy import select

from app.deps import DBSession
from app.models import User
from app.schemas import Token, UserCreate, UserOut
from app.security import create_access_token, hash_password, verify_password

router = APIRouter(prefix="/auth", tags=["auth"])


@router.post("/register", response_model=UserOut, status_code=status.HTTP_201_CREATED)
async def register(payload: UserCreate, db: DBSession):
    exists = await db.execute(select(User).where(User.username == payload.username))
    if exists.scalar_one_or_none():
        raise HTTPException(status.HTTP_409_CONFLICT, "Username already taken")

    user = User(
        username=payload.username,
        hashed_password=hash_password(payload.password),
    )
    db.add(user)
    await db.commit()
    await db.refresh(user)
    return user


@router.post("/token", response_model=Token)
async def login(
    form: Annotated[OAuth2PasswordRequestForm, Depends()],
    db: DBSession,
):
    result = await db.execute(select(User).where(User.username == form.username))
    user = result.scalar_one_or_none()

    if user is None or not verify_password(form.password, user.hashed_password):
        raise HTTPException(
            status.HTTP_401_UNAUTHORIZED,
            "Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )

    return Token(access_token=create_access_token(user.username))
```

### `app/routers/items.py`

```python
from typing import Annotated

from fastapi import APIRouter, HTTPException, Query, status
from sqlalchemy import select

from app.deps import ActiveUser, DBSession
from app.models import Item
from app.schemas import ItemCreate, ItemOut, ItemUpdate

router = APIRouter(prefix="/items", tags=["items"])


@router.get("", response_model=list[ItemOut])
async def list_items(
    db: DBSession,
    user: ActiveUser,
    skip: Annotated[int, Query(ge=0)] = 0,
    limit: Annotated[int, Query(ge=1, le=100)] = 20,
):
    result = await db.execute(
        select(Item)
        .where(Item.owner_id == user.id)
        .order_by(Item.id)
        .offset(skip)
        .limit(limit)
    )
    return list(result.scalars().all())


@router.get("/{item_id}", response_model=ItemOut)
async def get_item(item_id: int, db: DBSession, user: ActiveUser):
    item = await db.get(Item, item_id)
    # 404 (not 403) so we don't reveal that someone else's item exists
    if item is None or item.owner_id != user.id:
        raise HTTPException(status.HTTP_404_NOT_FOUND, "Item not found")
    return item


@router.post("", response_model=ItemOut, status_code=status.HTTP_201_CREATED)
async def create_item(payload: ItemCreate, db: DBSession, user: ActiveUser):
    item = Item(**payload.model_dump(), owner_id=user.id)
    db.add(item)
    await db.commit()
    await db.refresh(item)
    return item


@router.patch("/{item_id}", response_model=ItemOut)
async def update_item(
    item_id: int, payload: ItemUpdate, db: DBSession, user: ActiveUser,
):
    item = await db.get(Item, item_id)
    if item is None or item.owner_id != user.id:
        raise HTTPException(status.HTTP_404_NOT_FOUND, "Item not found")

    # exclude_unset so a partial PATCH only touches supplied fields
    for field, value in payload.model_dump(exclude_unset=True).items():
        setattr(item, field, value)

    await db.commit()
    await db.refresh(item)
    return item


@router.delete("/{item_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_item(item_id: int, db: DBSession, user: ActiveUser):
    item = await db.get(Item, item_id)
    if item is None or item.owner_id != user.id:
        raise HTTPException(status.HTTP_404_NOT_FOUND, "Item not found")
    await db.delete(item)
    await db.commit()
    return None
```

### `app/main.py`

```python
from contextlib import asynccontextmanager

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from app.config import get_settings
from app.database import Base, engine
from app.routers import auth, items

settings = get_settings()


@asynccontextmanager
async def lifespan(app: FastAPI):
    # Dev convenience only — use Alembic in production
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield
    await engine.dispose()


app = FastAPI(
    title=settings.app_name,
    version="1.0.0",
    lifespan=lifespan,
)

if settings.cors_origins:
    app.add_middleware(
        CORSMiddleware,
        allow_origins=settings.cors_origins,
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )

app.include_router(auth.router)
app.include_router(items.router)


@app.get("/health", include_in_schema=False)
async def health():
    return {"status": "ok"}
```

### `tests/conftest.py`

```python
import pytest
from fastapi.testclient import TestClient
from sqlalchemy.ext.asyncio import async_sessionmaker, create_async_engine
from sqlalchemy.pool import StaticPool

from app.database import Base, get_session
from app.main import app

TEST_URL = "sqlite+aiosqlite:///:memory:"


@pytest.fixture
def client():
    engine = create_async_engine(
        TEST_URL,
        connect_args={"check_same_thread": False},
        poolclass=StaticPool,          # one shared in-memory DB
    )
    factory = async_sessionmaker(engine, expire_on_commit=False)

    async def override_get_session():
        async with factory() as session:
            yield session

    async def setup():
        async with engine.begin() as conn:
            await conn.run_sync(Base.metadata.create_all)

    import anyio
    anyio.run(setup)

    app.dependency_overrides[get_session] = override_get_session
    with TestClient(app) as c:
        yield c
    app.dependency_overrides.clear()


@pytest.fixture
def auth_headers(client):
    client.post("/auth/register",
                json={"username": "alice", "password": "supersecret123"})
    r = client.post("/auth/token",
                    data={"username": "alice", "password": "supersecret123"})
    return {"Authorization": f"Bearer {r.json()['access_token']}"}
```

### `tests/test_items.py`

```python
def test_health(client):
    assert client.get("/health").json() == {"status": "ok"}


def test_requires_auth(client):
    assert client.get("/items").status_code == 401


def test_create_and_list(client, auth_headers):
    r = client.post(
        "/items",
        json={"name": "Widget", "price": 9.99},
        headers=auth_headers,
    )
    assert r.status_code == 201
    created = r.json()
    assert created["name"] == "Widget"
    assert created["price"] == 9.99

    r = client.get("/items", headers=auth_headers)
    assert r.status_code == 200
    assert len(r.json()) == 1
    assert r.json()[0]["id"] == created["id"]


def test_rejects_invalid_price(client, auth_headers):
    r = client.post(
        "/items", json={"name": "Bad", "price": -1}, headers=auth_headers,
    )
    assert r.status_code == 422


def test_patch_is_partial(client, auth_headers):
    item = client.post(
        "/items", json={"name": "A", "price": 1.0}, headers=auth_headers,
    ).json()

    r = client.patch(f"/items/{item['id']}", json={"price": 2.5}, headers=auth_headers)
    assert r.status_code == 200
    assert r.json()["name"] == "A"          # untouched
    assert r.json()["price"] == 2.5


def test_isolates_between_users(client, auth_headers):
    item = client.post(
        "/items", json={"name": "Mine", "price": 1.0}, headers=auth_headers,
    ).json()

    client.post("/auth/register",
                json={"username": "bob", "password": "anothersecret1"})
    bob = client.post("/auth/token",
                      data={"username": "bob", "password": "anothersecret1"})
    bob_headers = {"Authorization": f"Bearer {bob.json()['access_token']}"}

    # Bob must not see or fetch Alice's item
    assert client.get("/items", headers=bob_headers).json() == []
    assert client.get(f"/items/{item['id']}", headers=bob_headers).status_code == 404


def test_delete(client, auth_headers):
    item = client.post(
        "/items", json={"name": "Temp", "price": 1.0}, headers=auth_headers,
    ).json()
    assert client.delete(f"/items/{item['id']}", headers=auth_headers).status_code == 204
    assert client.get(f"/items/{item['id']}", headers=auth_headers).status_code == 404
```

### `requirements.txt`

```
fastapi[standard]
pydantic-settings
python-multipart
pyjwt
pwdlib[argon2]
sqlalchemy[asyncio]
aiosqlite
alembic
python-dotenv
pytest
anyio
httpx
```

### Run it

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
echo "SECRET_KEY=$(python -c 'import secrets; print(secrets.token_urlsafe(48))')" > .env

pytest -q                 # run the tests
fastapi dev app/main.py   # http://127.0.0.1:8000/docs

# Smoke test the flow
curl -X POST localhost:8000/auth/register \
  -H 'Content-Type: application/json' \
  -d '{"username":"alice","password":"supersecret123"}'

TOKEN=$(curl -s -X POST localhost:8000/auth/token \
  -d 'username=alice&password=supersecret123' | python -c 'import sys,json;print(json.load(sys.stdin)["access_token"])')

curl -X POST localhost:8000/items \
  -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' \
  -d '{"name":"Widget","price":9.99}'

curl -H "Authorization: Bearer $TOKEN" localhost:8000/items
```

**Note:** `sqlite+aiosqlite` keeps the example dependency-free, but SQLite's type affinity and concurrency behaviour differ from Postgres. Swap `DATABASE_URL` to `postgresql+asyncpg://...` for anything beyond a demo — the models and queries are unchanged.

## Appendix A: Pydantic v1 → v2 Cheat Sheet

| v1 | v2 |
|---|---|
| `@validator("x")` | `@field_validator("x")` + `@classmethod` |
| `@root_validator` | `@model_validator(mode="before" \| "after")` |
| `class Config: orm_mode = True` | `model_config = ConfigDict(from_attributes=True)` |
| `class Config: allow_population_by_field_name` | `ConfigDict(populate_by_name=True)` |
| `obj.dict()` | `obj.model_dump()` |
| `obj.json()` | `obj.model_dump_json()` |
| `Model.parse_obj(d)` | `Model.model_validate(d)` |
| `Model.parse_raw(s)` | `Model.model_validate_json(s)` |
| `Model.from_orm(o)` | `Model.model_validate(o, from_attributes=True)` |
| `Field(regex=...)` | `Field(pattern=...)` |
| `Field(const=True)` | `Field(frozen=True)` |
| `__fields__` | `model_fields` |
| `Config.schema_extra` | `ConfigDict(json_schema_extra=...)` |
| `validator(pre=True)` | `field_validator(mode="before")` |
| `@validator(always=True)` | `field_validator(mode="after")` with a default |
| `pydantic.BaseSettings` | `pydantic_settings.BaseSettings` (separate package) |
| `parse_obj_as(T, x)` | `TypeAdapter(T).validate_python(x)` |

## Appendix B: Dependency Injection Patterns

| Pattern | Shape | Use for |
|---|---|---|
| Function | `def dep(): ...` | Shared params, computed values |
| Factory | `def dep(cfg): return inner` | Configurable behaviour |
| Class | `class D: def __init__(self, a=1)` | Stateful config |
| Yield | `def dep(): yield res; cleanup()` | Sessions, files, locks |
| Async yield | `async def dep(): yield res` | Async resources |
| Sub-dependency | `Depends()` inside `Depends()` | Composition |
| Global | `FastAPI(dependencies=[...])` | App-wide guards |
| Router-level | `APIRouter(dependencies=[...])` | Group-wide guards |
| Route-level | `dependencies=[...]` on the decorator | Single-route guards |
| Override | `app.dependency_overrides[dep]` | Testing |
| No cache | `Depends(f, use_cache=False)` | Force re-evaluation |

## Appendix C: Response Payload Rules

| You return | FastAPI sends |
|---|---|
| `dict` / `list` | JSON |
| Pydantic model | JSON via `model_dump` |
| `str` | Plain text unless `response_class` says otherwise |
| `None` with 204 | Empty body |
| `Response` instance | Used directly, no serialisation |
| `JSONResponse(...)` | Used directly with your status/headers |
| Generator / async generator | Streamed **only** via `response_class=StreamingResponse`/`EventSourceResponse` or wrapped in one; a bare `return gen` is not streamed |
| `FileResponse(path)` | File bytes with range support |
| ORM model with `from_attributes` | Mapped fields only |
| ORM model without `from_attributes` | Serialisation error |

## Appendix D: Quick Reference Card

```python
# Imports worth knowing
from fastapi import (
    FastAPI, APIRouter, Depends, Security, Request, Response, BackgroundTasks,
    HTTPException, Query, Path, Body, Form, File, Header, Cookie,
    UploadFile, WebSocket, WebSocketDisconnect, status,
)
from fastapi.responses import (
    JSONResponse, ORJSONResponse, HTMLResponse, PlainTextResponse,
    RedirectResponse, StreamingResponse, FileResponse,
)
from fastapi.middleware.cors import CORSMiddleware
from fastapi.security import (
    OAuth2PasswordBearer, OAuth2PasswordRequestForm,
    HTTPBearer, HTTPBasic, APIKeyHeader,
)
from fastapi.testclient import TestClient


# The five things you do most
app = FastAPI(lifespan=lifespan)
router = APIRouter(prefix="/p", tags=["p"])
DBSession = Annotated[AsyncSession, Depends(get_session)]
@app.get("/x", response_model=Out, status_code=status.HTTP_200_OK)
raise HTTPException(status.HTTP_404_NOT_FOUND, "not found")


# Signature patterns
async def a(item_id: int):                                   # path
async def b(q: str | None = None, limit: int = 20):          # query
async def c(item: ItemIn):                                   # body
async def d(db: DBSession, user: CurrentUser):               # injected
async def e(response: Response):                             # headers/cookies
async def f(bg: BackgroundTasks):                            # post-response
async def g(request: Request):                               # raw request


# Partial update
for k, v in payload.model_dump(exclude_unset=True).items():
    setattr(obj, k, v)


# Timed middleware
@app.middleware("http")
async def timing(request: Request, call_next):
    r = await call_next(request)
    return r
```
