# Exception

```python
from typing import Any, Optional
from http import HTTPStatus

# Custom Base Exception
class BaseAppException(Exception):
    """Base exception for the application"""
    def __init__(self, message: str, status_code: int = HTTPStatus.INTERNAL_SERVER_ERROR):
        self.message = message
        self.status_code = status_code
        super().__init__(self.message)

# Data Validation Exceptions
class ValidationException(BaseAppException):
    """Raised when data validation fails"""
    def __init__(self, message: str):
        super().__init__(message, HTTPStatus.BAD_REQUEST)

class DataNotFoundException(BaseAppException):
    """Raised when requested data is not found"""
    def __init__(self, message: str):
        super().__init__(message, HTTPStatus.NOT_FOUND)

# API Exceptions
class APIException(BaseAppException):
    """Base exception for API related errors"""
    pass

class UnauthorizedException(APIException):
    """Raised when authentication fails"""
    def __init__(self, message: str = "Unauthorized access"):
        super().__init__(message, HTTPStatus.UNAUTHORIZED)

class ForbiddenException(APIException):
    """Raised when user doesn't have permission"""
    def __init__(self, message: str = "Access forbidden"):
        super().__init__(message, HTTPStatus.FORBIDDEN)

# Business Logic Exceptions
class BusinessLogicException(BaseAppException):
    """Base exception for business logic errors"""
    def __init__(self, message: str):
        super().__init__(message, HTTPStatus.UNPROCESSABLE_ENTITY)

class DuplicateEntryException(BusinessLogicException):
    """Raised when trying to create a duplicate entry"""
    pass

class InvalidOperationException(BusinessLogicException):
    """Raised when operation is not allowed"""
    pass

```

```python
from fastapi import FastAPI, Request, HTTPException
from fastapi.responses import JSONResponse
from typing import Optional, Dict, Any
from datetime import datetime

from exceptions import (
    BaseAppException,
    ValidationException,
    DataNotFoundException,
    UnauthorizedException,
    ForbiddenException,
    BusinessLogicException
)
from logger import logger
from models import UserCreate, UserResponse
from utils import format_response

app = FastAPI()

# Exception handlers
@app.exception_handler(BaseAppException)
async def base_exception_handler(request: Request, exc: BaseAppException) -> JSONResponse:
    logger.error(
        "Application exception occurred",
        extra={
            "error_type": exc.__class__.__name__,
            "error_message": exc.message,
            "status_code": exc.status_code,
            "path": request.url.path,
            "method": request.method
        }
    )
    return JSONResponse(
        status_code=exc.status_code,
        content=format_response(False, exc.message, None)
    )

@app.exception_handler(Exception)
async def general_exception_handler(request: Request, exc: Exception) -> JSONResponse:
    logger.error(
        "Unhandled exception occurred",
        extra={
            "error_type": exc.__class__.__name__,
            "error_message": str(exc),
            "path": request.url.path,
            "method": request.method
        }
    )
    return JSONResponse(
        status_code=500,
        content=format_response(False, "Internal server error", None)
    )

# Example service class with exception handling
class UserService:
    def __init__(self):
        self.users: Dict[int, Dict[str, Any]] = {}
        self.next_id = 1

    def validate_user_data(self, user_data: dict) -> None:
        """Validate user data"""
        if not user_data.get("email"):
            raise ValidationException("Email is required")
        
        if not isinstance(user_data.get("age"), int):
            raise ValidationException("Age must be an integer")
        
        if user_data.get("age") < 0:
            raise ValidationException("Age cannot be negative")

    def check_duplicate_email(self, email: str) -> None:
        """Check for duplicate email"""
        if any(user["email"] == email for user in self.users.values()):
            raise DuplicateEntryException(f"User with email {email} already exists")

    async def create_user(self, user_data: dict) -> dict:
        """Create a new user with exception handling"""
        try:
            # Validate user data
            self.validate_user_data(user_data)
            
            # Check for duplicate email
            self.check_duplicate_email(user_data["email"])
            
            # Create user
            user_id = self.next_id
            self.next_id += 1
            
            user = {
                "id": user_id,
                **user_data,
                "created_at": datetime.utcnow(),
                "updated_at": datetime.utcnow()
            }
            
            self.users[user_id] = user
            return user
            
        except ValidationException as e:
            logger.warning(
                "Validation error during user creation",
                extra={"error": str(e), "user_data": user_data}
            )
            raise
            
        except DuplicateEntryException as e:
            logger.warning(
                "Duplicate user creation attempted",
                extra={"error": str(e), "email": user_data.get("email")}
            )
            raise
            
        except Exception as e:
            logger.error(
                "Unexpected error during user creation",
                extra={"error": str(e), "user_data": user_data}
            )
            raise BusinessLogicException(f"Failed to create user: {str(e)}")

# API endpoints using the service
user_service = UserService()

@app.post("/users/", response_model=UserResponse)
async def create_user(user: UserCreate, request: Request):
    try:
        created_user = await user_service.create_user(user.dict())
        logger.info(
            "User created successfully",
            extra={"user_id": created_user["id"]}
        )
        return format_response(True, "User created successfully", created_user)
        
    except BaseAppException:
        # Let the exception handler handle it
        raise
        
    except Exception as e:
        # Log unexpected errors and convert to BaseAppException
        logger.error(
            "Unexpected error in create_user endpoint",
            extra={"error": str(e)}
        )
        raise BusinessLogicException("Failed to process request")

# Example middleware for authentication
from fastapi.middleware.base import BaseHTTPMiddleware

class AuthMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        # Check for auth token
        auth_token = request.headers.get("Authorization")
        if not auth_token:
            raise UnauthorizedException()
            
        try:
            # Validate token (example)
            if not self.validate_token(auth_token):
                raise ForbiddenException()
                
            response = await call_next(request)
            return response
            
        except BaseAppException:
            raise
        except Exception as e:
            logger.error(
                "Auth middleware error",
                extra={"error": str(e)}
            )
            raise UnauthorizedException()

    def validate_token(self, token: str) -> bool:
        # Implement token validation logic
        pass

# Add middleware
app.add_middleware(AuthMiddleware)
```

Key features of this exception handling approach:
1. Hierarchy of Exceptions:
- Base application exception
- Specific categories (Validation, API, Business Logic)
- Custom status codes and messages
2. Centralized Error Handling:
- Global exception handlers
- Consistent error response format
- Proper logging
3. Granular Exception Types:
- ValidationException for data validation
- DataNotFoundException for missing resources
- UnauthorizedException for auth issues
- BusinessLogicException for logic errors
4. Logging Integration:
- Detailed error logging
