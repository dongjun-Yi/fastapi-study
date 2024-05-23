# Pydantic models

`Pydantic`은 데이터 검증에 사용되는 파이썬 라이브러리다. **데이터 검증**을 통해 **의도치 않은 데이터 삽입을 방지하고** **예외처리**를 가능하게 하여 외부에서 들어온 **데이터를 안전하게 저장**할 수 있다.

```python
from pydantic import BaseModel, ValidationError

class Address(BaseModel):  # BaseModel 상속
  street: str
  city: str
  state: str
  postal_code: str

class User(BaseModel):
  id: int
  name: str
  email: str
  age: int | None
  addresses: list[Address] = []

# Valid data
user_data = {
    "id": 1,
    "name": "John Doe",
    "email": "johndoe@example.com",
    "age": 30,
    "addresses": [
        {
            "street": "123 Main St",
            "city": "Anytown",
            "state": "CA",
            "postal_code": "12345"
        }
    ]
}

try:
  user = User(**user_data)
  print(user)
except ValidationError as e:
  print(e.json())
```

위의 예제와 같이 `BaseModel`을 상속받아 `User 클래스`에 정의된 속성에 올바른 타입을 삽입하여 인스턴스를 생성하게 되면 요구사항에 의도한대로 데이터가 삽입되고 저장된다. 이처럼 `pydantic` 라이브러리를 이용해 데이터를 정의할 수 있다.

```python
# Invalid data
invalid_user_data = {
    "id": "one",
    "name": "John Doe",
    "email": "johndoe@example.com",
    "age": "thirty",
}

try:
    invalid_user = User(**invalid_user_data)
except ValidationError as e:
    print(e.json())
    
# [{"type":"int_parsing","loc":["id"],"msg":"Input should be a valid integer, 
# unable to parse string as an integer","input":"one",
# "url":"https://errors.pydantic.dev/2.7/v/int_parsing"},
# {"type":"int_parsing","loc":["age"],
# "msg":"Input should be a valid integer, unable to parse string as an integer",
# "input":"thirty","url":"https://errors.pydantic.dev/2.7/v/int_parsing"}]
```

만약 **클래스를 선언한대로 속성을 대입하지 못하면** **에러가 발생**하여 이에 맞게 적절히 예외처리를 구현해 프로그램에 안정성을 높일 수 있다.