# Path Parameters

### Path parameters with types

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

URL 파라미터는 라우터에 매핑되는 함수 매개변수에 **타입힌트**를 사용하여 **`int` 자료형**만 받을 수 있도록 한다.

만약 URL 파라미터에 `int`가 아닌 문자열로 요청하게 되면 **데이터 검증**에 의해 잘못된 요청임을 반환한다.

```json
// http://127.0.0.1:8000/items/foo로 요청 시 반환되는 응답 메세지
{
  "detail": [
    {
      "type": "int_parsing",
      "loc": [
        "path",
        "item_id"
      ],
      "msg": "Input should be a valid integer, unable to parse string as an integer",
      "input": "foo"
    }
  ]
}
```

### 순서 문제

URL 파라미터를 지정할 때, 가변인자가 아닌 고정된 경로일 경우 가변인자로 매핑되는 함수보다 위에 선언해야 한다.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}

@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}
```

`/users/me` 로 요청하게 되면 `{"user_id": "the current user"}`이 반환되고, 만약 r`ead_user_me()`가 `read_user(user_id: str)`보다 아래에 선언된다면 r`ead_user(user_id: str)`이 실행된다..

### **Predefined values**

만약 경로 매개변수에 따른 분기작업이 있을 경우 사전에 정의된 데이터로 판별해야 한다. 이때 enum을 이용하여 데이터를 정의할 수 있다.

```python
from enum import Enum

from fastapi import FastAPI

# Enum 클래스 정의
class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

app = FastAPI()

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name is ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}

    if model_name.value == "lenet":
        return {"model_name": model_name, "message": "LeCNN all the images"}

    return {"model_name": model_name, "message": "Have some residuals"}
```

`ModelName` 클래스는 `Enum`을 상속받아 정의하도록 하여 클래스안에 데이터는 문자열 타입으로 정의한다. 또한 str 클래스를 상속받아 구현하여 문자열 타입에도 해당된다.

경로 매개변수에 `ModelName` 타입으로 문자열을 받고, `Enum` 클래스 안에 정의된 문자열을 기준으로 분기하여 각각 다른 응답을 보내도록 할 수 있다. 만약 `Enum` 클래스안에 정의된 문자열이 아닌 다른 문자열로 요청하면 잘못된 요청으로 응답을 반환하게 된다.

```json
// http://127.0.0.1:8000/models/abc
{
  "detail": [
    {
      "type": "enum",
      "loc": [
        "path",
        "model_name"
      ],
      "msg": "Input should be 'alexnet', 'resnet' or 'lenet'",
      "input": "abc",
      "ctx": {
        "expected": "'alexnet', 'resnet' or 'lenet'"
      }
    }
  ]
}
```