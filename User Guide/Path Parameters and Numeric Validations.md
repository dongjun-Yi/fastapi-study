# Path Parameters and Numeric Validations

### **Declare metadata**

경로 매개변수에 대한 설명이 필요할 경우 `Path()`를 이용해 경로 파라미터에 대한 주석을 남길 수 있다.

```python
from typing import Annotated
from fastapi import FastAPI, Path, Query

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get")],
    q: Annotated[str | None, Query(alias="item-query")] = None,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

### **Number validations: greater than and less than or equal**

경로 매개변수의 숫자 검증은 `gt`(greater than)과 `le`(less than)로 범위를 지정하여 데이터의 범위를 제한할 수 있다.

```python
from typing import Annotated
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get", gt=0, le=1000)],
    q: str,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

위 코드는 `Path()`를 이용해 0보다 크고 1000보다 작은 int 정수형만 받도록 경로 매개변수에 대한 데이터 검증을 수행할 수 있다.