# Coroutine

`코루틴(coroutine)`이란 제어권이 실행하는 **다른 함수와 대등한 관계**를 가지고 동작하는 함수를 말한다. 여기서 중요한 키워드는 **제어권**인데, 이 제어권을 설명하기 위해 일반 함수를 통해 알아보자.

```python
def add(a,b):
		c = a + b
		print(c)
		print('add 함수')
 
def calc():
    add(1, 2)    # add 함수가 끝나면 다시 calc 함수로 돌아옴
    print('calc 함수')
 
calc()
```

이 코드는  `calc()`가 `add()`함수를 호출하는데, 이때 main routine은 `calc()`, `add()`는 sub routine이다. 이 함수들의 동작과정을 그림으로 나타내면 아래와 같다.

![Untitled](Coroutine%202edde4ec00d24e69a5df5e3a6ba3ba54/Untitled.png)

메인 루틴에서 서브 루틴을 실행하면 서브루틴을 실행 후 다시 메인 루틴으로 돌아온다. 이때, 서브 루틴의 내용은 사라지고, 서브 루틴은 메인 루틴에 종속적으로 존재하게 된다. 즉, **제어권은 메인 루틴에 존재**하게 된다.
이와 달리 코루틴은 메인 루틴과 **대등한 관계를 가져** 코드를 실행하는데 **제어권을 동등**하게 가지게 된다.

![Untitled](Coroutine%202edde4ec00d24e69a5df5e3a6ba3ba54/Untitled%201.png)

위의 그림에서 메인 루틴과 코루틴이 번갈아가면서 실행하게 되는데, 서로 다른 코드를 실행하게 되면 대기 상태로 전환되고 그 동안 다른 코드를 실행 하는 것이다. 이 코루틴으로 파이썬에서는 **동시성 프로그래밍이 가능**해져, 한 명령어가 시간이 많이 걸리는 작업이 있을 경우, 코루틴의 대기 상태에서 다른 코드를 실행해 **CPU 활용률**을 높일 수 있다.

코루틴을 이제 코드로 한번 알아보자.

### Synchronous Code

```python
import time

def brewCoffee():
  print("Start brewCoffee()")
  time.sleep(3)
  print("End brewCoffee()")
  return "Coffee ready"

def toastBagel():
  print("Start toastBagel()")
  time.sleep(2)
  print("End toastBagel()")
  return "Bagel toasted"
```

```python
def main():
  start_time = time.time()

  result_coffee = brewCoffee()
  result_bagel = toastBagel()

  end_time = time.time()

  elasped_time = end_time - start_time
  print(f"Result of brewCoffee : {result_coffee}")
  print(f"Result of toastBagel : {result_bagel}")
  print(f"Total execution time : {elasped_time:.2f} seconds")

if __name__ == "__main__":
  main()
```

```python
Start brewCoffee()
End brewCoffee()
Start toastBagel()
End toastBagel()
Result of brewCoffee : Coffee ready
Result of toastBagel : Bagel toasted
Total execution time : 5.01 seconds
```

이 코드는 커피를 만드는 함수와 토스트를 만드는 함수를 **동기적으로 실행한 코드**다. 이코드는 커피를 다 만들고 끝난 후 토스트를 만드는 함수를 실행한다. 근데 이 2개의 함수를 순차적으로 실행하는 것이 아닌 같이 실행하게 되면 안될까?

### Asynchronous Code

```python
import asyncio
import time

async def brewCoffee():
  print("Start brewCoffee()")
  await asyncio.sleep(3)
  print("End brewCoffee()")
  return "Coffee ready"

async def toastBagel():
  print("Start toastBagel()")
  await asyncio.sleep(2)
  print("End toastBagel()")
  return "Bagel toasted"
```

```python
async def main():
  start_time = time.time()

  batch = asyncio.gather(brewCoffee(), toastBagel())
  result_coffee, result_bagel = await batch

  end_time = time.time()

  elasped_time = end_time - start_time
  print(f"Result of brewCoffee : {result_coffee}")
  print(f"Result of toastBagel : {result_bagel}")
  print(f"Total execution time : {elasped_time:.2f} seconds")

if __name__ == "__main__":
  asyncio.run(main())
```

```python
Start brewCoffee()
Start toastBagel()
End toastBagel()
End brewCoffee()
Result of brewCoffee : Coffee ready
Result of toastBagel : Bagel toasted
Total execution time : 3.00 seconds
```

위의 순차적으로 실행한 함수들을 `async` 함수로 선언하고 `await asyncio.sleep()`을 실행하여 비동기적으로 실행하였더니 이전의 5초가 걸렸던 코드가 **3초**로 줄어들었다. 이는 비동기적으로 실행했기 때문에 `sleep()` 명령어에서 외부 IO 작업에 기다리지 않고 다른 함수를 실행시켜 **CPU의 활용률을 높인 것**이다. 
이는 코루틴 덕분에 함수를 `중단(Suspend)`하고 `재개(Resume)`할 수 있어 동시성을 활용할 수 있었다.

또한 코루틴의 개수를 활용하여 **비동기식 코드**를 작성할 수 있다. 위의 코드에서는 `asyncio.gather()`를 이용해 함수 2개를 여러 코루틴에서 실행하게 하였고, `asyncio.create_task()`를 통해 2개의 task를 생성해 하나의 코루틴에서 실행하도록 할 수 있다.

### main

```python
async def main():
  start_time = time.time()

  # batch = asyncio.gather(brewCoffee(), toastBagel())
  # result_coffee, result_bagel = await batch

  coffee_task = asyncio.create_task(brewCoffee())
  bagel_task = asyncio.create_task(toastBagel())

  result_coffee = await coffee_task
  result_bagel = await bagel_task

  end_time = time.time()

  elasped_time = end_time - start_time
  print(f"Result of brewCoffee : {result_coffee}")
  print(f"Result of toastBagel : {result_bagel}")
  print(f"Total execution time : {elasped_time:.2f} seconds")
 
```

```python
Start brewCoffee()
Start toastBagel()
End toastBagel()
End brewCoffee()
Result of brewCoffee : Coffee ready
Result of toastBagel : Bagel toasted
Total execution time : 3.00 seconds
```

`asyncio.gather()` 와 `asyncio.create_task()` 의 실행속도는 똑같이 나왔다. 성능 상 큰 차이는 없는 걸로 보이고 개별적으로 관리하려는 작업이 있으면 `asyncio.create_task()`를, 여러 코루틴의 결과를 집계하는 데는 `asyncio.gather()` 를 사용하는 것이 유리하다.