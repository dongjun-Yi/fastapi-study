# Body - Multiple Parameters

### **Multiple body parameters**

메세지 Body에 담긴 데이터들을 여러개의 데이터로 보내도 받을 수 있다.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

class User(BaseModel):
    username: str
    full_name: str | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User):
    results = {"item_id": item_id, "item": item, "user": user}
    return results
```

<img width="743" alt="Untitled" src="https://github.com/dongjun-Yi/fastapi-study/assets/90665186/a03afa73-e3d5-4ff6-be5f-d70e4077c348">


<img width="762" alt="Untitled 1" src="https://github.com/dongjun-Yi/fastapi-study/assets/90665186/cbc262d1-c23a-46d6-8c54-7ef6ad6e2972">


### **Embed a single body parameter**

만약 Body 데이터 부분이 반드시 하나 이상 존재하고 Body 매개변수에 `embed=True`를 넣어 FastAPI는 바로 Body 데이터를 예측할 수 있게 된다.

```python
from typing import Annotated
from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Annotated[Item, Body(embed=True)]):
    results = {"item_id": item_id, "item": item}
    return results
```

위의 `PUT` 메서드의 경로수행 함수에 대한 클라이언트에의 요청 JSON 데이터는 아래와 같이 된다.

```json
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    }
}

// Not Like This
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2
}
```

`item` 객체 안에 속성들을 구성하여 보내면 `embed=true` 속서에 의해 데이터를 받을 수 있다.
