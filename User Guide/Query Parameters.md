# QueryParameters

경로 매개변수에 포함되지 않은 매개변수들은 쿼리 파라미터로 간주한다.

```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

위와 같은 로직으로 구성된 애플리케이션에 [http://127.0.0.1:8000/items/?skip=1&limit=2](http://127.0.0.1:8000/items/?skip=1&limit=2)을 요청하게 되면 `list[1]` ~ `list[2]`의 값을 반환하게 된다.

```json
[
  {
    "item_name": "Bar"
  },
  {
    "item_name": "Baz"
  }
]
```

만약 쿼리 파라미터를 생략해서 요청하게 되면 default 값으로 함수 매개변수에 `skip = 0`, `limit = 10`으로 정의했기 때문에 default 값으로 설정되어 응답을 반환하게 된다.

```json
// http://127.0.0.1:8000/items/
[
  {
    "item_name": "Foo"
  },
  {
    "item_name": "Bar"
  },
  {
    "item_name": "Baz"
  }
]
```

### 다중 경로 파라미터와 쿼리 파라미터

URL 경로에 여러 파라미터들을 사용하여 매핑할 수 있다.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(
    user_id: int, item_id: str, q: str | None = None, short: bool = False
):
    item = {"item_id": item_id, "owner_id": user_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```

### **Required query parameters**

쿼리 매개변수에 타입 힌트를 지정하면 **필수 매개변수**로 요청시 **반드시 해당 속성과 값과 함께 요청**해야 한다.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_user_item(item_id: str, needy: str):
    item = {"item_id": item_id, "needy": needy}
    return item
```

만약 [http://127.0.0.1:8000/items/foo-item](http://127.0.0.1:8000/items/foo-item)로 전송하게 되면 `needy` 값이 없기 때문에 에러가 발생한다.

```json
{
  "detail": [
    {
      "type": "missing",
      "loc": [
        "query",
        "needy"
      ],
      "msg": "Field required",
      "input": null,
      "url": "https://errors.pydantic.dev/2.1/v/missing"
    }
  ]
}
```

아래 3개의 경로 파리미터와 쿼리 파리미터를 포함한 라우터를 살펴보자.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_user_item(
    item_id: str, needy: str, skip: int = 0, limit: int | None = None
):
    item = {"item_id": item_id, "needy": needy, "skip": skip, "limit": limit}
    return item
```

- `item_id` : 문자열의 필수 파라미터
- `needy` : 문자열의 필수 파라미터
- `skip` : 정수형의 생략 가능한 파라미터
- `limit` : 정수형과 생략 가능한 파라미터