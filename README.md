from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Dict

app = FastAPI()

db: Dict[int, dict] = {}


class User(BaseModel):
    id: int
    FirstName: str
    LastName: str
    isMale: bool


@app.post("/users/", response_model=User)
async def create_user(user: User):
    if user.id in db:
        raise HTTPException(status_code=400, detail="User ID already exists")
    user_dict = user.dict()
    db[user.id] = user_dict
    return user

@app.get("/users/{user_id}", response_model=User)
async def read_user(user_id: int):
    if user_id not in db:
        raise HTTPException(status_code=404, detail="User not found")
    return db[user_id]


@app.put("/users/{user_id}", response_model=User)
async def update_user(user_id: int, user: User):
    if user_id not in db:
        raise HTTPException(status_code=404, detail="User not found")
    db[user_id] = user.dict()
    return db[user_id]


@app.delete("/users/{user_id}", response_model=User)
async def delete_user(user_id: int):
    if user_id not in db:
        raise HTTPException(status_code=404, detail="User not found")
    deleted_user = db.pop(user_id)
    return deleted_user
