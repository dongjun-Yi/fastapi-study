# Declare Request Example Data

### **Extra JSON Schema data in Pydantic models**

경로 수행 함수에 맞는 JSON 데이터 포맷을 정의하여 문서화를 진행할 수 있다.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

    class Config:
        schema_extra = {
            "examples": [
                {
                    "name": "Foo",
                    "description": "A very nice Item",
                    "price": 35.4,
                    "tax": 3.2,
                }
            ]
        }

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

`Item` 클래스안에 `Config` 클래스를 정의하여 `schema_extra`에 JSON 요청 예시를 작성하면 Swagger 문서에 요청에 대한 예시를 자동으로 작성해준다.

### **examples in JSON Schema - OpenAPI**

요청한 Body에 JSON 포맷을 정의하게 되면 swagger docs에 자동으로 응답 예제로 작성된다.

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
async def update_item(
    item_id: int,
    item: Annotated[
        Item,
        Body(
            examples=[
                {
                    "name": "Foo",
                    "description": "A very nice Item",
                    "price": 35.4,
                    "tax": 3.2,
                }
            ],
        ),
    ],
):
    results = {"item_id": item_id, "item": item}
    return results
```

<img width="1444" alt="Untitled" src="https://github.com/dongjun-Yi/fastapi-study/assets/90665186/4a84284c-db8c-4967-bd01-a021daf7e8f3">


## **OpenAPI-specific examples**

Swagger API 문서에 각 예시마다 요청에 대한 JSON 포맷을 작성하여 편리하게 문서화를 할 수 있다.

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
async def update_item(
    *,
    item_id: int,
    item: Annotated[
        Item,
        Body(
            openapi_examples={
                "normal": {
                    "summary": "A normal example",
                    "description": "A **normal** item works correctly.",
                    "value": {
                        "name": "Foo",
                        "description": "A very nice Item",
                        "price": 35.4,
                        "tax": 3.2,
                    },
                },
                "converted": {
                    "summary": "An example with converted data",
                    "description": "FastAPI can convert price `strings` to actual `numbers` automatically",
                    "value": {
                        "name": "Bar",
                        "price": "35.4",
                    },
                },
                "invalid": {
                    "summary": "Invalid data is rejected with an error",
                    "value": {
                        "name": "Baz",
                        "price": "thirty five point four",
                    },
                },
            },
        ),
    ],
):
    results = {"item_id": item_id, "item": item}
    return results
```

**Request Examples on Swagger**

<img width="1421" alt="Untitled 1" src="https://github.com/dongjun-Yi/fastapi-study/assets/90665186/a53e0ec1-1603-49c8-8b95-371fa64a1bd1">
