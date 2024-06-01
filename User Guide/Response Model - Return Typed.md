# Response Model - Return Typed

요청에 대한 응답으로 파이썬 **타입 힌트를 사용**하여 반환되는 데이터 타입을 명시할 수 있다.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: list[str] = []

@app.post("/items/")
async def create_item(item: Item) -> Item:
    return item

@app.get("/items/")
async def read_items() -> list[Item]:
    return [
        Item(name="Portal Gun", price=42.0),
        Item(name="Plumbus", price=32.0),
    ]
```

이렇게 응답에 대한 타입을 명시하면 output 데이터를 제한하여 필터할 수 있게 되고, 속성을 외부로 노출시키지 않을 수 있어 보안적인 측면에도 좋다.

**POST /items**

<img width="757" alt="Untitled" src="https://github.com/dongjun-Yi/fastapi-study/assets/90665186/a33d7b35-03ab-49be-9236-f838e77209c7">


**GET /items**

<img width="757" alt="Untitled 1" src="https://github.com/dongjun-Yi/fastapi-study/assets/90665186/404c1760-787c-4850-aa71-e5a06eb9dd96">


타입 힌트 대신 `response_model`로 타입을 명시할 수 있다.

```python
from typing import Any

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: list[str] = []

@app.post("/items/", response_model=Item)
async def create_item(item: Item) -> Any:
    return item

@app.get("/items/", response_model=list[Item])
async def read_items() -> Any:
    return [
        {"name": "Portal Gun", "price": 42.0},
        {"name": "Plumbus", "price": 32.0},
    ]
```

### **response_model Priority**

만약 반환타입으로 타입힌트와 `response_model` 을 같이 선언했다면, `response_model` 이 더 **우선순위가 높아** `response_model` 로 정의한 반환타입으로 동작하게 된다.

### **Return Type and Data Filtering**

만약 회원가입에 대한 요청을 처리하고 응답하는 기능을 만든다고 해보자. 이때, 비밀번호와 같은 민감한 정보는 절대 평문으로 output으로 보내면 안된다. 보안상의 이유로 데이터를 외부로부터 감추어야 할때, 아래와 같이 코드를 작성할 수 있다.

```python
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()

class BaseUser(BaseModel):
    username: str
    email: EmailStr
    full_name: str | None = None

class UserIn(BaseUser):
    password: str

@app.post("/user/")
async def create_user(user: UserIn) -> BaseUser:
    return user
```

Input 데이터 모델로는 `password` 속성만 존재하고 `BaseUser`를 상속받은 `UserIn` 타입으로 받고, 응답할 때는 비밀번호를 제외한 `BaseUser` 타입의 객체를 반환하여 보안상의 이슈를 해결할 수 있다.

<img width="755" alt="Untitled 2" src="https://github.com/dongjun-Yi/fastapi-study/assets/90665186/17bde236-0816-4852-b580-9ba8b29dbf88">


### **Return a Response Directly**

JSON 포맷의 응답이 아닌 다른 URL로 이동하는 `RedirectResponse` 객체를 이용하여 응답을 반환할 수 있다.

```python
from fastapi import FastAPI
from fastapi.responses import RedirectResponse

app = FastAPI()

@app.get("/teleport")
async def get_teleport() -> RedirectResponse:
    return RedirectResponse(url="https://www.youtube.com/watch?v=dQw4w9WgXcQ")
```

### **Response Model encoding parameters**

반환하는 데이터 모델에 대해 **필수 항목이 아닌 필드들을 제외**하여 응답을 반환하고 싶을 경우 
`response_model_exclude_unset=True` 속성을 추가해주면 된다.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float = 10.5
    tags: list[str] = []

items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}

@app.get("/items/{item_id}", response_model=Item, response_model_exclude_unset=True)
async def read_item(item_id: str):
    return items[item_id]
```

```json
// http://127.0.0.1:8000/item/foo
{
    "name": "Foo",
    "price": 50.2
}
```
