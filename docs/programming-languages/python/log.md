# Loguru

Output json format log with loguru

```python
import sys
import json
from datetime import datetime
from loguru import logger
from typing import Any, Dict, Union

class JsonSink:
    """Custom sink for JSON formatted logs"""
    
    def __init__(self):
        self.serialize = json.dumps

    def write(self, message: Dict[str, Any]) -> None:
        """
        Write message as JSON to stdout
        """
        serialized = self.serialize({
            "timestamp": message.record["time"].isoformat(),
            "level": message.record["level"].name,
            "message": message.record["message"],
            "logger_name": "fastapi_app",
            **message.record["extra"]  # Include any extra fields
        })
        sys.stdout.write(serialized + "\n")

def setup_logger() -> None:
    """Configure loguru logger with JSON formatting"""
    
    # Remove default handler
    logger.remove()
    
    # Add JSON handler
    logger.add(
        JsonSink().write,
        serialize=True,
        level="INFO"
    )

# Initialize logger
setup_logger()

# Export logger instance
app_logger = logger
```

```python
from fastapi import FastAPI, Request, HTTPException
from fastapi.responses import JSONResponse
from fastapi.middleware.base import BaseHTTPMiddleware
from typing import Callable
import time
from uuid import uuid4

from logger import app_logger
from models import UserCreate, UserUpdate, UserResponse, UsersListResponse, DeleteResponse
from exceptions import BaseAppException
from utils import format_response

class LoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next: Callable):
        # Generate request ID
        request_id = request.headers.get("X-Request-ID", str(uuid4()))
        
        # Start timing
        start_time = time.time()
        
        # Prepare context
        with app_logger.contextualize(
            request_id=request_id,
            method=request.method,
            path=request.url.path,
            client_ip=request.client.host,
            user_agent=request.headers.get("user-agent")
        ):
            # Log request
            app_logger.info("Request started")
            
            try:
                response = await call_next(request)
                
                # Calculate process time
                process_time = time.time() - start_time
                
                # Log response
                app_logger.info(
                    "Request completed",
                    status_code=response.status_code,
                    process_time=f"{process_time:.3f}s"
                )
                
                return response
                
            except Exception as e:
                # Log error
                process_time = time.time() - start_time
                app_logger.error(
                    "Request failed",
                    error=str(e),
                    process_time=f"{process_time:.3f}s"
                )
                raise

app = FastAPI()
app.add_middleware(LoggingMiddleware)

@app.exception_handler(BaseAppException)
async def app_exception_handler(request: Request, exc: BaseAppException):
    app_logger.warning(
        exc.message,
        error_type=exc.__class__.__name__,
        status_code=exc.status_code
    )
    return JSONResponse(
        status_code=exc.status_code,
        content=format_response(False, exc.message, None)
    )

@app.exception_handler(Exception)
async def general_exception_handler(request: Request, exc: Exception):
    app_logger.error(
        "Unhandled exception",
        error_type=exc.__class__.__name__,
        error_message=str(exc)
    )
    return JSONResponse(
        status_code=500,
        content=format_response(False, "Internal server error", None)
    )

@app.post(
    "/users/",
    response_model=UserResponse,
    status_code=201,
    tags=["users"]
)
async def create_user(user: UserCreate, request: Request):
    try:
        # Log operation start with context
        with app_logger.contextualize(operation="create_user"):
            app_logger.info(
                "Creating new user",
                user_email=user.email
            )
            
            created_user = await db.create_user(user.dict())
            
            # Log success
            app_logger.info(
                "User created successfully",
                user_id=created_user["id"]
            )
            
            return format_response(True, "User created successfully", created_user)
            
    except Exception as e:
        # Log error with context
        app_logger.error(
            "Failed to create user",
            error=str(e),
            user_email=user.email
        )
        raise

@app.get(
    "/users/{user_id}",
    response_model=UserResponse,
    tags=["users"]
)
async def get_user(user_id: int, request: Request):
    with app_logger.contextualize(
        operation="get_user",
        user_id=user_id
    ):
        try:
            app_logger.info("Fetching user")
            
            user = await db.get_user_by_id(user_id)
            if not user:
                app_logger.warning("User not found")
                raise HTTPException(status_code=404, detail="User not found")
                
            app_logger.info("User retrieved successfully")
            return format_response(True, "User retrieved successfully", user)
            
        except Exception as e:
            app_logger.error("Failed to fetch user", error=str(e))
            raise