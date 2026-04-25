# FastAPI 知识体系完整总结

## 目录
- [框架概述](#框架概述)
- [核心架构](#核心架构)
- [安装与配置](#安装与配置)
- [基本组件](#基本组件)
- [路由系统](#路由系统)
- [请求与响应](#请求与响应)
- [数据验证](#数据验证)
- [依赖注入](#依赖注入)
- [安全认证](#安全认证)
- [数据库集成](#数据库集成)
- [高级特性](#高级特性)
- [测试与部署](#测试与部署)

## 框架概述

### 什么是 FastAPI
```python
"""
FastAPI 是一个现代、快速（高性能）的 Web 框架，用于基于标准 Python 类型提示构建 API
"""
```

### 主要特性
- ⚡ **高性能**：与 NodeJS 和 Go 相当
- 🚀 **快速开发**：开发速度提高约 200%-300%
- 📝 **简洁直观**：强大的编辑器支持，自动补全
- 🔒 **健壮安全**：自带数据验证、序列化、文档
- 📚 **标准兼容**：基于开放标准（OpenAPI、JSON Schema）

### 适用场景
- RESTful API 开发
- 微服务架构
- 实时应用程序（WebSocket）
- 数据密集型应用
- 机器学习模型服务化

## 核心架构

### 技术栈组成
```
FastAPI = Starlette (Web框架) + Pydantic (数据验证) + 类型提示
```

### 架构层次
1. **Starlette** - 处理 Web 底层功能
2. **Pydantic** - 处理数据验证和序列化
3. **FastAPI** - 集成层和高级功能

### 请求生命周期
```python
客户端请求 → 路由匹配 → 依赖注入 → 数据验证 → 业务逻辑 → 响应序列化 → 客户端响应
```

## 安装与配置

### 基础安装
```bash
pip install fastapi
pip install uvicorn[standard]
```

### 最小应用示例
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

### 启动服务
```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

## 基本组件

### FastAPI 实例
```python
from fastapi import FastAPI

app = FastAPI(
    title="My API",
    description="An amazing API",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)
```

### 路径操作装饰器
```python
@app.get("/")        # GET 请求
@app.post("/")       # POST 请求  
@app.put("/")        # PUT 请求
@app.delete("/")     # DELETE 请求
@app.patch("/")      # PATCH 请求
@app.options("/")    # OPTIONS 请求
@app.head("/")       # HEAD 请求
```

### 路径操作函数
```python
@app.get("/items/{item_id}")
async def read_item(item_id: int):  # 支持 async/await
    return {"item_id": item_id}

@app.get("/items/")
def read_items():  # 也支持同步函数
    return ["item1", "item2"]
```

## 路由系统

### 路径参数
```python
@app.get("/items/{item_id}")
def get_item(item_id: int):  # 类型自动转换和验证
    return {"item_id": item_id}

@app.get("/users/{user_id}/items/{item_id}")
def get_user_item(user_id: int, item_id: str):
    return {"user_id": user_id, "item_id": item_id}
```

### 预定义路径参数
```python
from enum import Enum

class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    return {"model_name": model_name}
```

### 查询参数
```python
@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10, q: str = None):
    return {"skip": skip, "limit": limit, "q": q}
```

### 路由分组 - APIRouter
```python
from fastapi import APIRouter

router = APIRouter()

@router.get("/users/")
def read_users():
    return ["user1", "user2"]

@router.post("/users/")
def create_user(user: User):
    return user

# 在主应用中包含路由
app.include_router(router, prefix="/api/v1", tags=["users"])
```

## 请求与响应

### 请求体 - Pydantic 模型
```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None

@app.post("/items/")
def create_item(item: Item):
    return item
```

### 响应模型
```python
@app.post("/items/", response_model=Item)
def create_item(item: Item):
    return item  # 自动过滤响应字段

# 排除默认值字段
@app.get("/items/", response_model=Item, response_model_exclude_unset=True)
def read_items():
    return items
```

### 响应状态码
```python
from fastapi import status

@app.post("/items/", status_code=status.HTTP_201_CREATED)
def create_item(item: Item):
    return {"id": 1, **item.dict()}
```

### 自定义响应
```python
from fastapi.responses import JSONResponse, HTMLResponse

@app.get("/custom/")
def custom_response():
    return JSONResponse(
        content={"message": "Hello"},
        status_code=201,
        headers={"X-Custom-Header": "value"}
    )

@app.get("/html/", response_class=HTMLResponse)
def read_html():
    return "<h1>Hello World</h1>"
```

## 数据验证

### Pydantic 字段验证
```python
from pydantic import BaseModel, Field, validator

class Item(BaseModel):
    name: str = Field(..., min_length=1, max_length=50)
    price: float = Field(..., gt=0)
    tax: float = Field(0.0, ge=0)
    
    @validator('name')
    def name_must_contain_space(cls, v):
        if ' ' not in v:
            raise ValueError('must contain a space')
        return v
```

### 路径参数验证
```python
from fastapi import Path

@app.get("/items/{item_id}")
def read_item(
    item_id: int = Path(..., title="The ID of the item", ge=1),
    q: str = Query(None, alias="item-query")
):
    return {"item_id": item_id, "q": q}
```

### 查询参数验证
```python
from fastapi import Query

@app.get("/items/")
def read_items(
    q: str = Query(
        None,
        min_length=3,
        max_length=50,
        regex="^[a-zA-Z0-9 ]*$"
    ),
    size: int = Query(1, ge=1, le=100)
):
    return {"q": q, "size": size}
```

## 依赖注入

### 基础依赖
```python
from fastapi import Depends

def common_parameters(q: str = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

@app.get("/items/")
def read_items(commons: dict = Depends(common_parameters)):
    return commons
```

### 类作为依赖
```python
class QueryParams:
    def __init__(self, q: str = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit

@app.get("/items/")
def read_items(params: QueryParams = Depends()):
    return params
```

### 数据库会话依赖
```python
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/users/{user_id}")
def get_user(user_id: int, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.id == user_id).first()
    return user
```

### 依赖缓存
```python
def get_heavy_computation():
    # 这个结果会被缓存
    return expensive_computation()

@app.get("/items/")
def read_items(computation: str = Depends(get_heavy_computation)):
    return {"result": computation}
```

## 安全认证

### OAuth2 密码流
```python
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.get("/users/me")
def read_current_user(token: str = Depends(oauth2_scheme)):
    user = decode_token(token)
    return user
```

### JWT 认证
```python
from jose import JWTError, jwt
from fastapi.security import HTTPBearer

security = HTTPBearer()

def verify_token(credentials: HTTPAuthorizationCredentials = Depends(security)):
    try:
        payload = jwt.decode(
            credentials.credentials, 
            SECRET_KEY, 
            algorithms=[ALGORITHM]
        )
        return payload
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")

@app.get("/protected")
def protected_route(user: dict = Depends(verify_token)):
    return user
```

### 完整的认证系统
```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBasic, HTTPBasicCredentials

security = HTTPBasic()

def authenticate(credentials: HTTPBasicCredentials = Depends(security)):
    correct_username = secrets.compare_digest(credentials.username, "admin")
    correct_password = secrets.compare_digest(credentials.password, "password")
    if not (correct_username and correct_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Basic"},
        )
    return credentials.username
```

## 数据库集成

### SQLAlchemy 集成
```python
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    hashed_password = Column(String)

# 依赖注入
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### 异步数据库支持
```python
from databases import Database

database = Database(DATABASE_URL)

@app.on_event("startup")
async def startup():
    await database.connect()

@app.on_event("shutdown")
async def shutdown():
    await database.disconnect()
```

### Tortoise-ORM (异步)
```python
from tortoise import fields
from tortoise.models import Model
from tortoise.contrib.fastapi import register_tortoise

class User(Model):
    id = fields.IntField(pk=True)
    email = fields.CharField(50, unique=True)
    
    class Meta:
        table = "users"

register_tortoise(
    app,
    db_url="sqlite://db.sqlite3",
    modules={"models": ["app.models"]},
    generate_schemas=True,
    add_exception_handlers=True,
)
```

## 高级特性

### 后台任务
```python
from fastapi import BackgroundTasks

def write_notification(email: str, message=""):
    # 模拟发送邮件
    with open("log.txt", mode="w") as email_file:
        content = f"notification for {email}: {message}"
        email_file.write(content)

@app.post("/send-notification/{email}")
def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_notification, email, message="some notification")
    return {"message": "Notification sent in the background"}
```

### WebSocket 支持
```python
from fastapi import WebSocket

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```

### 中间件
```python
import time
from fastapi import Request

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

### CORS 中间件
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # React 应用地址
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### 静态文件服务
```python
from fastapi.staticfiles import StaticFiles

app.mount("/static", StaticFiles(directory="static"), name="static")
```

### 自定义异常处理器
```python
from fastapi import Request
from fastapi.responses import JSONResponse

class CustomException(Exception):
    def __init__(self, name: str):
        self.name = name

@app.exception_handler(CustomException)
async def custom_exception_handler(request: Request, exc: CustomException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something."},
    )
```

### 事件处理
```python
@app.on_event("startup")
async def startup_event():
    print("Application starting up...")

@app.on_event("shutdown")
async def shutdown_event():
    print("Application shutting down...")
```

## 测试与部署

### 测试客户端
```python
from fastapi.testclient import TestClient

client = TestClient(app)

def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"Hello": "World"}
```

### 异步测试
```python
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_read_item():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        response = await ac.get("/items/1")
    assert response.status_code == 200
```

### 部署配置

#### Uvicorn 配置
```python
# uvicorn_config.py
import uvicorn

if __name__ == "__main__":
    uvicorn.run(
        "main:app",
        host="0.0.0.0",
        port=8000,
        reload=True,  # 开发环境
        workers=4,    # 生产环境
    )
```

#### Docker 部署
```dockerfile
FROM python:3.9

WORKDIR /code

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "80"]
```

#### 环境配置
```python
from pydantic import BaseSettings

class Settings(BaseSettings):
    app_name: str = "My API"
    database_url: str
    secret_key: str
    
    class Config:
        env_file = ".env"

settings = Settings()
```

### 性能优化技巧

1. **使用异步操作**
```python
@app.get("/")
async def read_data():
    data = await database.fetch_all(query)
    return data
```

2. **合理使用依赖缓存**
```python
@lru_cache()
def get_expensive_data():
    return expensive_operation()
```

3. **启用 Gzip 压缩**
```python
from fastapi.middleware.gzip import GZipMiddleware
app.add_middleware(GZipMiddleware, minimum_size=1000)
```

## 总结

FastAPI 的核心优势：

- ✅ **类型安全**：基于 Python 类型提示
- ✅ **自动文档**：交互式 API 文档
- ✅ **高性能**：媲美 NodeJS 和 Go
- ✅ **易于学习**：直观的 API 设计
- ✅ **生产就绪**：包含所有必要功能
- ✅ **标准兼容**：基于 OpenAPI 和 JSON Schema

这个总结涵盖了 FastAPI 的所有核心概念和功能，从基础的路由定义到高级的安全认证和数据库集成，为构建生产级别的 API 提供了完整的知识体系。