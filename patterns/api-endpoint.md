# Pattern: FastAPI Endpoint

## Standard CRUD Endpoint Structure

```python
# backend/app/api/v1/{resource}.py
from fastapi import APIRouter, Depends, HTTPException, status
from uuid import UUID
from typing import List

from app.core.supabase import get_supabase
from app.core.auth import get_current_user
from app.models.schemas import ResourceCreate, Resource, ResourceUpdate

router = APIRouter(prefix="/{resources}", tags=["{resources}"])


@router.get("", response_model=List[Resource])
async def list_resources(
    user = Depends(get_current_user),
    supabase = Depends(get_supabase)
):
    """List all resources for the current user."""
    response = supabase.table("{resources}") \
        .select("*") \
        .eq("user_id", user.id) \
        .order("created_at", desc=True) \
        .execute()

    return response.data


@router.get("/{id}", response_model=Resource)
async def get_resource(
    id: UUID,
    user = Depends(get_current_user),
    supabase = Depends(get_supabase)
):
    """Get a single resource by ID."""
    response = supabase.table("{resources}") \
        .select("*") \
        .eq("id", str(id)) \
        .eq("user_id", user.id) \
        .single() \
        .execute()

    if not response.data:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Resource not found"
        )

    return response.data


@router.post("", response_model=Resource, status_code=status.HTTP_201_CREATED)
async def create_resource(
    data: ResourceCreate,
    user = Depends(get_current_user),
    supabase = Depends(get_supabase)
):
    """Create a new resource."""
    response = supabase.table("{resources}") \
        .insert({
            **data.model_dump(),
            "user_id": user.id
        }) \
        .execute()

    return response.data[0]


@router.patch("/{id}", response_model=Resource)
async def update_resource(
    id: UUID,
    data: ResourceUpdate,
    user = Depends(get_current_user),
    supabase = Depends(get_supabase)
):
    """Update a resource."""
    # First verify ownership
    existing = supabase.table("{resources}") \
        .select("id") \
        .eq("id", str(id)) \
        .eq("user_id", user.id) \
        .single() \
        .execute()

    if not existing.data:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Resource not found"
        )

    response = supabase.table("{resources}") \
        .update(data.model_dump(exclude_unset=True)) \
        .eq("id", str(id)) \
        .execute()

    return response.data[0]


@router.delete("/{id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_resource(
    id: UUID,
    user = Depends(get_current_user),
    supabase = Depends(get_supabase)
):
    """Delete a resource."""
    response = supabase.table("{resources}") \
        .delete() \
        .eq("id", str(id)) \
        .eq("user_id", user.id) \
        .execute()

    if not response.data:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Resource not found"
        )

    return None
```

## Auth Dependency

```python
# backend/app/core/auth.py
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import jwt, JWTError
from app.core.config import settings

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security)
):
    """Validate JWT and return user info."""
    try:
        payload = jwt.decode(
            credentials.credentials,
            settings.SUPABASE_JWT_SECRET,
            algorithms=["HS256"],
            audience="authenticated"
        )
        return type('User', (), {
            'id': payload.get('sub'),
            'email': payload.get('email')
        })()
    except JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid token"
        )
```

## Registering Routes

```python
# backend/app/main.py
from app.api.v1 import clients, fields, scripts, recommendations

app.include_router(clients.router, prefix="/api/v1")
app.include_router(fields.router, prefix="/api/v1")
app.include_router(scripts.router, prefix="/api/v1")
app.include_router(recommendations.router, prefix="/api/v1")
```

## Error Response Format

```python
from fastapi import HTTPException

# Use consistent error format
raise HTTPException(
    status_code=400,
    detail={
        "code": "VALIDATION_ERROR",
        "message": "Invalid crop type",
        "field": "crop"
    }
)
```
