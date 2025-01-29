<h1>
  <span class="headline">FastAPI Serializers and Controllers</span>
  <span class="subhead">Connecting Controllers to Database</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to implement real CRUD operations by adding serializers and connecting the FastAPI controllers to a database.

## Connecting our API

In this section, we will walk through the steps required to add serializers and connect our API to a database.

1. Import all necessary models and schemas into each controller module.
2. Use the correct **ModelSchema** for each CRUD route.
3. Add error handling to each controller function.
4. Test each endpoint (index, detail, create, delete, and update) using FastAPI’s built-in documentation.

## Getting started

We will focus on the `teas` controller, which is located in the `controllers/teas.py` file.

### 1. Import necessary modules

First, let's ensure we have all the required imports at the top of the file. Edit `controllers/teas.py` to look like this:

```py

# controllers/teas.py

from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from models.tea import TeaModel
from serializers.tea import TeaSchema
from typing import List
from database import get_db

router = APIRouter()
```

## Updating the Tea controller

In this section, we will implement CRUD operations by refactoring the routes for handling `GET`, `POST`, `PUT`, `PATCH`, and `DELETE` HTTP methods.

| Method   | CRUD Functionality | Database Action      |
| -------- | ------------------ | -------------------- |
| `GET`    | Read               | Retrieve data        |
| `POST`   | Create             | Add new data         |
| `PUT`    | Update             | Modify existing data |
| `PATCH`  | Update             | Modify existing data |
| `DELETE` | Delete             | Remove data          |

### 1. `Read/GET` All Teas Route

Let’s refactor the route to get all the teas. This is the `GET /teas` endpoint:

```py
@router.get("/teas", response_model=List[TeaSchema])
def get_teas(db: Session = Depends(get_db)):
    teas = db.query(TeaModel).all()
    return teas
```

- `GET /teas` retrieves a list of all teas from the database.
- We use the `TeaSchema` Pydantic model to serialize the database results into a format that can be sent as JSON.

### 2. `Read/GET` Single Tea Route

Next, let’s refactor the route to get a single tea. This is the `GET /teas/{tea_id}` endpoint:

```py
@router.get("/teas/{tea_id}", response_model=TeaSchema)
def get_single_tea(tea_id: int, db: Session = Depends(get_db)):
    tea = db.query(TeaModel).filter(TeaModel.id == tea_id).first()
    if not tea:
        raise HTTPException(status_code=404, detail="Tea not found")
    return tea
```

- `GET /teas/{tea_id}` retrieves a single tea by its `id`.
- If no tea with the given `tea_id` is found, we raise a 404 error with a message saying `"Tea not found"`.

### 3. Create/POST Route

Now, let’s refactor the `POST /teas` route to create a new tea in the database:

```py
@router.post("/teas", response_model=TeaSchema)
def create_tea(tea: TeaSchema, db: Session = Depends(get_db)):
    new_tea = TeaModel(**tea.dict()) # Convert Pydantic model to SQLAlchemy model
    db.add(new_tea)
    db.commit() # Save to database
    db.refresh(new_tea) # Refresh to get the updated data (including auto-generated fields)
    return new_tea
```

- `POST /teas` creates a new tea entry.
- The `tea.dict()` converts the Pydantic model into a format that can be used to create a SQLAlchemy model.
- The `**` syntax unpacks the dictionary returned by `tea.dict()` into keyword arguments when passing it to the TeaModel constructor

### 4. Update/PUT Route

Let’s now refactor the `PUT /teas/{tea_id}` route to update an existing tea:

```py
@router.put("/teas/{tea_id}", response_model=TeaSchema)
def update_tea(tea_id: int, tea: TeaSchema, db: Session = Depends(get_db)):
    db_tea = db.query(TeaModel).filter(TeaModel.id == tea_id).first()
    if not db_tea:
        raise HTTPException(status_code=404, detail="Tea not found")

    tea_data = tea.dict(exclude_unset=True)  # Only update the fields provided
    for key, value in tea_data.items():
        setattr(db_tea, key, value)

    db.commit()  # Save changes
    db.refresh(db_tea)  # Refresh to get updated data
    return db_tea
```

- `PUT /teas/{tea_id}` updates an existing tea.
- We use the `tea.dict(exclude_unset=True)` method to only update the fields that have been modified, leaving other fields unchanged.

---

### 5. DELETE Route

Finally, let’s refactor the `DELETE /teas/{tea_id}` route to delete a tea:

```py
@router.delete("/teas/{tea_id}")
def delete_tea(tea_id: int, db: Session = Depends(get_db)):
    db_tea = db.query(TeaModel).filter(TeaModel.id == tea_id).first()
    if not db_tea:
        raise HTTPException(status_code=404, detail="Tea not found")

    db.delete(db_tea)  # Remove from database
    db.commit()  # Save changes
    return {"message": f"Tea with ID {tea_id} has been deleted"}
```

- `DELETE /teas/{tea_id}` removes a tea from the database.
- If the tea is not found, we raise a 404 error with the message `"Tea not found"`.

## Verify All Endpoints

To test the API routes, run your FastAPI application with:

```bash
pipenv run uvicorn main:app --reload
```

Now you can test each endpoint using FastAPI's built-in documentation.

> Navigate to FastAPI Documentation: Open [`http://localhost:8000/docs`](http://localhost:8000/docs) in your browser.
