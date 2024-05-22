# Python Types Intro

파이썬에서 **타입 힌트**란 함수의 매개변수나 함수의 return **값의 타입을 명시**하는 기능이다. 파이썬은 동적 프로그래밍 언어이기 때문에 실행 시점에 타입이 결정되어 메모리 공간에 타입에 따라 메모리가 할당된다. 
따라서 파이썬을 사용하는 코더들은 코드로만 보왔을 때 함수의 매개변수나 반환 값에 타입을 알 수 없어 코드 가독성이 떨어진다. **타입 힌트는** 이를 해결하기 위한 기능으로, 실행시키기 전에 변수나 함수에 **타입을 명시해** **런타임 전에 타입 체크를 가능하게 해** 오류를 방지할 수 있다.

### python without type hint

```python
def get_full_name(first_name, last_name):
  full_name = first_name.title() + " " + last_name.title()
  return full_name

print(get_full_name("john", "doe")) # John, Doe
```

위의 함수에서 매개변수로 타입에 대한 명시 없이 코드를 작성하면 이 코드는 어떤 변수의 타입을 넘겨하는지 협업하는 개발자끼리 혼란을 일으킬 수 있다. 따라서 여기에 다른 정적 프로그래밍 언어와 같이 타입을 명시한다면 코드의 가독성을 향상시킬 수 있다.

### python with type hint

```python
def get_full_name(first_name: str, last_name: str) -> str:
  full_name = first_name.title() + " " + last_name.title()
  return full_name
  
  print(get_full_name("john", ))
  # Argument of type "Literal[1]" cannot be assigned to parameter 
  # "last_name" of type "str" in function "get_full_name"
```

함수의 매개변수에 타입힌트를 명시하고, 호출측에서 매개변수의 타입을 다르게 삽입하게 되면 에러가 발생되며 코드를 실행하게 되면 `AttributeError: 'int' object has no attribute 'title’` 이 발생하게 된다. 중요한 것은 파이썬의 **타입 힌트는 런타임 시에는 적용이 되지 않고**, 단지 **주석에 불과한 것**을 알아야 한다.

## Generics Types with type Parameters

---

파이썬은 `dict, list, set tuple`과 같은 자료구조는 서로 다른 값을 저장할 수 있다. 
이러한 내부에 저장된 데이터 타입을 `generic`라 하며 내부에 저장된 값, 즉 `generic` type을 지정할 수 있다.

```python
# List와 Tuple의 generic 타입 지정
def process_items(items_t: tuple[int, int, str], items_s: list[str]):
		return items_t, items_s

# dict의 generic 타입 지정
def process_items(prices: dict[str, float]):
    for item_name, item_price in prices.items():
        print(item_name)
        print(item_price)
```

위와 같이 매개변수에 `list, tuple, dict`를 받고, 타입 힌트로 내부에 저장될 값을 명시하여 나타낼 수 있다. 

### Union

변수의 타입을 여러개 받을 때 union을 사용하고, python 3.10 버전에서는 |을 사용하여 union 기능을 

```python
def process_item(item: int | str):
  print(item)
```

### Possibly None

`None` 값이 매개변수로 전달되는 오류를 방지하기 위해 `Optional`  type hint를 사용할 수 있다.

```python
def say_hi(name: Optional[str] = None):
    if name is not None:
        print(f"Hey {name}!")
    else:
        print("Hello World")
        
# say_hi() => Argument missing for parameter "name"
say_hi(name=None)
```

위와 같이 Optional로 타입힌트를 사용하고, 함수를 빈 인자로 호출하면 에러가 발생하게 된다. python 3.10버전부터는 Optional 키워드 없이 |을 사용해 None 타입을 명시하면 된다.

```python
def say_hi(name: str | None):
  if name is not None:
    print(name)
  else:
    print("Hello World")
```

### **Classes as types**

매개변수 인자로 클래스 타입을 타입힌트로 사용할 수 있다.

```python
class Person:
  def __init__(self, name: str):
    self.name = name

def get_perosn_name(person: Person):
  return person.name

```