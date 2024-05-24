# Concurrency and async/await

## Aysncronous code

---

비동기 코드는 **다른 작업**이 다른 곳에서 **완료될 때까지 기다려야 함을 알리는 방법**이 있음을 의미한다. 여기서 기다리는 작업은 데이터베이스 쿼리, API 전송과 같은 `IO Bound` 연산을 말한다.
비동기는 느린 작업에 대해 계속 기다리지 않고 다른 연산을 수행하다가 멈춰놨던 작업이 끝나게 되면 그때 맞춰서 연산을 다시 수행하는 작업을 말한다.

### **Concurrency** and **parallelism**

`동시성과 병령성`은 유사한것 같지만 다른 개념이다. 이 둘의 차이를 알기 위해 햄버거를 사는 과정으로 알아보자.

![Untitled](https://github.com/dongjun-Yi/fastapi-study/assets/90665186/ff99fa0e-cc5b-40d1-ad55-08bcda0f9a20)


손님은 햄버거를 사려고 카운터에 주문한 다음 손님들은 자리를 고르고 얘기를 하며 다른 일을 할 수 있다. 그리고 주문이 나오게 되면 그때 햄버거를 가지고 와 먹을 수 있다.

![Untitled 1](https://github.com/dongjun-Yi/fastapi-study/assets/90665186/ba939eb0-971c-41eb-a217-b40c9a68d6af)


위의 과정이 `동시성에 대한 예시`이다. 햄버거를 조리하는 동안 손님들은 다른 작업을 수행하고 햄버거가 완성되면 그때 맞춰서 작업을 하여 기다리는 작업이 있으면 다른 일을 수행하는 것이다.

![Untitled 2](https://github.com/dongjun-Yi/fastapi-study/assets/90665186/2477a199-abdf-4f6b-bc3b-f299a32c3fff)


이번에는 `병령성에 대한 예시`이다. 병렬성은 먼저 카운터에 여러명이 존재하고 햄버거를 주문하게 되면 그 자리에서 계속 기다린다.

![Untitled 3](https://github.com/dongjun-Yi/fastapi-study/assets/90665186/5f21ded5-1c0e-4094-a97f-c68b6bd5a34b)

![Untitled 4](https://github.com/dongjun-Yi/fastapi-study/assets/90665186/c8832067-5578-467c-90a3-f1579a2b4c48)


기다린 후 햄버거가 완성되면 햄버거를 받고 그제서야 먹을 수 있게 된다. 이것이 병렬성에 대한 예시이며, 한 작업에 대해 기다리는 작업이 있으면 다른일을 하는 것이 아닌 작업이 끝날때까지 기다리는 작업을 말한다.

정리하면 동시성은 하나의 CPU에 여러 프로세스/스레드를 실행하는 것이고, 병렬성은 여러 CPU가 각각 하나의 프로세스/스레드를 실행하는 개념이다. 병렬성은 그래서 하나의 스레드만 실행하기 때문에 외부 IO 연산이 많아지면 느려지게 되며, 비동기성을 가지게 되면 높은 성능을 얻을 수 있게 된다.

## async/await

---

`async/await` 키워드는 비동기식으로 동작하는 코드를 순차적으로 동작하게 끔 비동기 코드를 정의하는 직관적인 방법이다. 

아래 `async`로 정의한 함수를 살펴보자.

```python
async def get_burgers(number: int):
    # Do some asynchronous stuff to create the burgers
    return burgers

# This won't work, because get_burgers was defined with: async def
burgers = get_burgers(2)
```

async로 정의한 함수는 비동기적인 작업을 수행하는 함수를 의미하며 이 코드를 실행시키게 되면 
`RuntimeWarning: coroutine 'get_burgers' was never awaited` 오류가 발생한다. 그 이유는 async로 정의한 함수는 `coroutine`이 되며 이 작업에서는 기다리는 연산이 있어야 하므로 오류가 발생한 것이다. 

### async with await function

```python
async def read_burgers():
  burgers = await get_burgers(2)
  return burgers
```

위와 같이 `async` 함수의 비동기적인 작업을 명시할 경우, `await` 키워드로 기다리는 연산을 수행해 코드를 수행할 수 있다.
