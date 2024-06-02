# User Sign up Scenario

유저가 회원가입을 하는 기능을 개발해보자. 이때 필요한 데이터 모델은 다음과 같다.

- **input model**
- **output model**
- **database model**

위의 모델들은 각각 필요에 따라 구분되었으며 각 모델마다 데이터의 제한사항은 다음과 같다.

- input model : 패스워드 데이터를 받아야 한다.
- output model : 패스워드를 노출해서는 안된다.
- database model : 데이터베이스에 해시된 값으로 저장해야 한다.

위의 데이터 모델로 기능을 만들게 되면 아래의 코드와 같다.

```python
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()

class UserBase(BaseModel):
    username: str
    email: EmailStr
    full_name: str | None = None

class UserIn(UserBase):
    password: str

class UserOut(UserBase):
    pass

class UserInDB(UserBase):
    hashed_password: str

def fake_password_hasher(raw_password: str):
    return "supersecret" + raw_password

def fake_save_user(user_in: UserIn):
    hashed_password = fake_password_hasher(user_in.password)
    user_in_db = UserInDB(**user_in.dict(), hashed_password=hashed_password)
    print("User saved! ..not really")
    return user_in_db

@app.post("/user/", response_model=UserOut)
async def create_user(user_in: UserIn):
    user_saved = fake_save_user(user_in)
    return user_saved
```

각각의 데이터모델과 함수를 살펴보면 다음과 같다.

- `UserBase(BaseModel)` : 유저에 대한 정보를 담은 데이터 모델
- `UserIn(UserBase)` : 유저 Input에서 password 속성을 포함한 데이터 모델
- `UserOut(UserBase)` : password를 제외한 유저에게 반환할 데이터 모델
- `UserInDB(UserBase)` : 해시된 비밀번호로 DB에 저장하기 위한 데이터 모델
- `fake_password_hasher()` : 비밀번호 해싱 함수
- `fake_save_user(user_in: UserIn)` : 해시된 비밀번호와 사용자 정보를 저장할 데이터 모델

위의 시나리오 처럼 각각의 용도에 맞춰서 데이터를 정의하고 기능에 맞게 활용할 수 있다.