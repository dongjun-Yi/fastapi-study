# Request Body

클라이언트에서 데이터를 보내 애플리케이션에 자원을 저장하는 행위는 HTTP `POST` 메서드로 메세지 Body에 데이터를 전송하게 된다. 그럼 fastapi는 엔드포인트에 따라 `pydantic` 의 `BaseModel` 상속받아 데이터를 저장할 수 있다.

```python
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.post("/items/")
async def create_item(item: Item):
    return item
```

위의 `Item` 클래스는 `BaseModel`을 상속받아 구현하였고, 이때 타입힌트를 통해 필수 인자로 `name, description, price, tax` 모두 지정한 것을 확인할 수 있다.
또한 경로 수행 함수에서는 매개변수로 `Item` 타입의 변수를 받아 메세지 Body 데이터를 확인할 수 있다.

<img width="755" alt="Untitled" src="https://github.com/dongjun-Yi/fastapi-study/assets/90665186/9e8a7f2a-2b7d-4ef2-b2d0-fe0aca011c36">

### **Request body + path + query parameters examples**

```python
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, q: str | None = None):
    result = {"item_id": item_id, **item.dict()}
    if q:
        result.update({"q": q})
    return result
```

위의 예제는 클라이언트의 `PUT` 메서드로 아이템을 업데이트 하는 기능이다. 이때 경로 매개변수, 쿼리 파라미터, 메세지 바디의 내용을 혼합하여 사용한 코드이다.

`update_item(item_id: int, item: Item, q: str | None = None):` 

- `item_id` : 경로 매개변수
- `item` : 메세지 Body의 데이터를 받는 BaseModel
- `q` : 쿼리 파라미터
