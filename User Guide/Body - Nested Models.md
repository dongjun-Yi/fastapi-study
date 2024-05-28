# Body - Nested Models

### **List fields**

`BaseModel`로 상속받은 클래스는 `list`를 이용하여 중첩된 모델 구조로 데이터를 저장할 수 있다.

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

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

<img width="758" alt="Untitled" src="https://github.com/dongjun-Yi/fastapi-study/assets/90665186/b961be66-a23c-4382-9c1e-51aeccdcd3cc">

<img width="757" alt="Untitled 1" src="https://github.com/dongjun-Yi/fastapi-study/assets/90665186/a14c4ce2-13cd-439e-829a-83b8dd3432f1">


### **Define a submodel**

중첩된 데이터 구조에서 `BaseModel` 속성안에 `BaseModel`로 정의한 데이터를 정의할 수 있다.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Image(BaseModel):
    url: str
    name: str

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()
    image: Image | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

위의 `Item` 데이터 모델에 image 속성은 `Image` 데이터 모델 타입으로 정의하여 중첩된 데이터 모델 구조로 정의한것을 확인할 수 있다.

### **Special types and validation**

`pydantic`에서 제공하는 데이터 타입으로 검증을 수행할 수 있다. 예를 들어 URL의 경우 HTTPURL을 사용하여 `str`인 문자열 타입 대신 검증을 할 수 있다.

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()

class Image(BaseModel):
    url: HttpUrl # instead of str
    name: str

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: set[str] = set()
    image: Union[Image, None] = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```
