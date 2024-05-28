# Query Parameters and String Validations

### **Additional validation**

쿼리 파라미터에 50글자 이상 넘지 않도록 데이터를 제한 시키려면 `Query`와 `Annontated`를 사용하면 된다.

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(max_length=50)] = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

```json
// http://127.0.0.1:8000/items/?q=zxcxbczxbczxbczhxbczhjxbczjhxbcjzhxbcjzhbcxchbzxhjcbzxjhcbzjhxcbzjhxbcjzhxbcjzhbxcbhzxc
{
  "detail": [
    {
      "type": "string_too_long",
      "loc": [
        "query",
        "q"
      ],
      "msg": "String should have at most 50 characters",
      "input": "zxcxbczxbczxbczhxbczhjxbczjhxbcjzhxbcjzhbcxchbzxhjcbzxjhcbzjhxcbzjhxbcjzhxbcjzhbxcbhzxc",
      "ctx": {
        "max_length": 50
      }
    }
  ]
}
```

쿼리 파라미터에 50글자 이상으로 작성하여 요청 시 `Query(max_length=50)`로 검증하여 에러가 발생해 응답을 반환하는 것을 확인할 수 있다.

### Annotated

`Annotated`는 변수나 함수의 반환값에 타입 유추가 가능하게 하는 메타데이터이다. `Annotated`를 통해 매개변수의 추가적인 검증을 할 수 있다.

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(max_length=50)] = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

`q: Annotated[str | None, Query(max_length=50)] = None`는 `Annotated`를 통해 쿼리 파라미터는 문자열의 타입으로 받고, 50글자 이상 넘지 않도록 제한할 수 있다. 또한 `= None`으로 생략 가능한 쿼리 파라미터이다.

### Default Values

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str, Query(min_length=3)] = "fixedquery"):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

만약 `Annotated[] = “default”` 는 쿼리 파라미터가 생략된 경우 “”안에 지정한 문자열로 초기화 되어 응답하게 된다.

### **Make it required**

만약 쿼리 파라미터에 필수값으로 지정하려면 `=`를 생략하여 작성하면 된다.

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str, Query(min_length=3)]):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

```json
// http://127.0.0.1:8000/items/
{
  "detail": [
    {
      "type": "missing",
      "loc": [
        "query",
        "q"
      ],
      "msg": "Field required",
      "input": null
    }
  ]
}
```

### **Required with Ellipsis (…)**

```python
from fastapi import FastAPI, Query
from typing_extensions import Annotated

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str, Query(min_length=3)] = ...):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

만약 필수 쿼리 파라미터를 지정할 때 `…`로 지정하게 되면 검증된 에러가 발생하는 것이 아닌 서버에 에러가 발생하게 된다.

```python
Internal Server Error
```

### **Query parameter list / multiple values**

쿼리 파라미터에 한 key값에 대해 여러 속성값을 `list`를 이용해 요청을 받을 수 있다.

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[list[str] | None, Query()] = None):
    query_items = {"q": q}
    return query_items
```

```json
// http://localhost:8000/items/?q=foo&q=bar
{
  "q": [
    "foo",
    "bar"
  ]
}
```

### **Query parameter list / multiple values with defaults**

쿼리 파라미터에서 다중 값에 대한 default 값 설정도 마찬가지로 `list`형태로 선언해 초기화할 수 있다.

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[list[str], Query()] = ["foo", "bar"]):
    query_items = {"q": q}
    return query_items
```

### **Declare more metadata**

쿼리 파라미터에 대한 주석을 추가할 수 있어 어떤 파라미터(`title`)인지, 그리고 그에 대한 설명(`description`)을 추가할 수 있다.

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    q: Annotated[
        str | None,
        Query(
            title="Query string",
            description="Query string for the items to search in the database that have a good match",
            min_length=3,
        ),
    ] = None,
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

`title`과 `description`을 이용해 쿼리 파라미터에 대한 정보를 나타낼 수 있다.

### **Alias parameters**

쿼리 파라미터에 대해 key값에 별칭을 설정하여 `alias`에 설정한 문자열에 대해 쿼리 파라미터를 작성하도록 제한할 수 있다.

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(alias="item-query")] = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

```json
// http://127.0.0.1:8000/items/?item-query=foobaritems
{
  "items": [
    {
      "item_id": "Foo"
    },
    {
      "item_id": "Bar"
    }
  ],
  "q": "foobaritems"
}
```