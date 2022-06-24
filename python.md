# Concept

> [Python Docs](https://docs.python.org/ko/3/) 
>
> [Global Interpreter Lock (GIL)](https://timewizhan.tistory.com/entry/Global-Interpreter-Lock-GIL?category=1028951) 
>
> [python 코딩 도장](https://dojang.io/course/view.php?id=7) 



## unpacking

`*` 을 list, tuple, dict 등의 앞에 붙이면 unpacking 해준다. 이렇게 된다는 얘기다.

```python
print(*[1,2]) # 1 2
print(*[[1,2],[3,4]]) # [1, 2] [3, 4]
```

예를 들어 아래와 같이 실행하면 에러가 난다. 당연하다. zip에 의해 a가 풀리면서 i가 `([1, 2],), ([3, 4],)` 로 나올텐데 list를 요소로 가진 tuple에 대해 sum을 했으니 될리가 없다.

```python
a = [[1,2], [3,4]]
print([sum(i) for i in zip(a)])

"""
Traceback (most recent call last):
  File "/Users/dgdsingen/repo/me/test/python/tmp/t.py", line 2, in <module>
    print([sum(i) for i in zip(a)])
  File "/Users/dgdsingen/repo/me/test/python/tmp/t.py", line 2, in <listcomp>
    print([sum(i) for i in zip(a)])
TypeError: unsupported operand type(s) for +: 'int' and 'list'
"""
```

아래와 같이 unpacking을 해주면 zip(*a) 결과로 `(1, 3), (2, 4)` 가 나오게 되고 sum이 잘 된다.

```python
a = [[1,2], [3,4]]
print([sum(i) for i in zip(*a)])
```



# Concurrent, Parallel

- Concurrent(동시성): 1대의 디바이스에 2개의 큐를 가지고 번갈아가며 요청을 처리하는 것을 생각하면 된다. 
- Parallel(병렬성): 2대의 디바이스에 각각 큐를 가지고 각각 요청을 처리하는 것을 생각하면 된다. 



> [Python Concurrency Programming](https://newsight.tistory.com/298) 
>
> [Python 동시 프로그래밍 – Che1's Blog](https://nachwon.github.io/asyncio-futures/) 
>
> [asyncio 를 활용한 동시성 - 1. threading 과의 비교 – Che1's Blog](https://nachwon.github.io/asyncio-and-threading/) 
>
> [multiprocessing vs multithreading vs asyncio in Python 3 - Stack Overflow](https://stackoverflow.com/questions/27435284/multiprocessing-vs-multithreading-vs-asyncio-in-python-3) 
>
> [Multiprocessing VS Threading VS AsyncIO in Python](https://ivdevlog.tistory.com/3) 



## multiprocessing, threading, asyncio

Python에서의 동시성 처리를 위해서는 우선 작업이 CPU Bound인지 IO Bound인지 파악한다. 

- CPU Bound : 작업 시간이 주로 CPU에 할당되는 것을 의미한다. 인코딩과 같은 작업이 이에 해당한다. 
- I/O Bound : 작업 시간이 주로 I/O에 할당되는 것을 의미한다. 대부분의 웹 서비스는 Request/Response 기반이므로 I/O Bound에 해당한다.



그 다음으로는 Python의 GIL(Global Interpreter Lock)을 고려한다. 

GIL은 말 그대로 전역 Lock이기 때문에 이에 의해 동시에 1개의 thread만 실행된다. (실제 물리적 CPU 코어 1개만 사용)

다만 이는 CPU Bound 작업일때의 이야기이고 I/O Bound 작업시에는 GIL이 해제되어 multi threading이 가능하다. 



그러므로 CPU Bound 작업을 한다면 multiprocessing을 사용하여 멀티 코어를 활용하는 것이 성능상 도움이 된다. 다만 이 경우 Python 프로세스를 여러개 구동하므로 프로세스를 만드는 오버헤드, 메모리 사용률 증가, 프로세스 간 통신 비용도 고려하자. 

반대로 I/O Bound 작업이라면 threading이 유용할 수 있다. I/O를 기다리며 idle 상태인 CPU에 threading을 사용하면 성능 향상을 볼 수 있다. 다만 theading은 메모리 공유에 유의해야 하고 프로세스보다는 덜하지만 다소 오버헤드가 있다는 점을 고려하자. 

그러나 asyncio를 사용하면 single thread + Non-blocking I/O 기반으로 동작하기 때문에 multi thread에 의한 오버헤드 없이 더 빠른 I/O Bound 작업이 가능하다. 예를 들어 2개의 request를 처리하는데 thread를 각각 2개 만들어 사용할 경우, thread 1은 request 1이 끝날때까지 idle 상태가 된다. (다른 할일이 없다) thread 1이 request 1을 기다렸다가 response를 받으면 이후의 thread 생명주기에 들어간다. asyncio를 사용하면 애초에 thread를 만들지 않았으니 thread 생명주기 관리가 없으며, request 1 시작 후 idle 상태 대신 바로 request 2로 넘어가고 event loop에 의해 response 1/2를 받아 처리하게 된다. request ~ response 사이 시간은 single thread에서 나머지 처리에 사용된다. 

정리하면 아래와 같다.

```python
if cpu_bound:
    multiprocessing
elif io_bound:
    if fast_io and limited_connections:
        threading
    elif slow_io and many_connections:
        asyncio

"""
fast_io and limited_connections 라고 해서 asyncio를 안쓰는 것이 아니다.
어차피 위 조건이면 threading을 쓰든 asyncio를 쓰든 크게 상관이 없다.
threading은 약간의 오버헤드가 있으니 가능한 한 threading 보다는 asyncio를 사용하자.
"""
```



## multiprocessing

Pool을 사용해 정해진 process 개수를 유지하면서 실행할 수 있다. 예를 들어 processes=2 인 경우 func1, func2가 2개가 병렬 실행되며 둘 중 하나가 종료되면 이어서 func3 실행됨.

Process를 사용해서 1개씩 띄울 수도 있다. 

```python
from multiprocessing import Process, Pool
import subprocess

def main1():
    with Pool(processes=2) as pool:
        for func in [func1, func2, func3]:
            pool.apply_async(func=func)
        pool.close()
        pool.join()

def main2():
    for func in [func1, func2]:
        Process(target=func).start()
```



multiprocessing 으로 다시 subprocess 를 띄우는 경우, Process가 죽든 말든 subprocess는 백그라운드에서 따로 돈다. 

```python
from multiprocessing import Process
import subprocess

def main():
    subprocess.Popen(["sleep", "10"])

if __name__ == '__main__':
    Process(target=main).start()
```



만약 subprocess 종료를 기다려야 한다면 아래와 같이 실행한다.

```python
from multiprocessing import Process
import subprocess

def main():
    p = subprocess.Popen(["sleep", "10"])
    p.wait()

if __name__ == '__main__':
    p = Process(target=main)
    p.start()
    p.join()
```



## asyncio

> [asyncio : 단일 스레드 기반의 Nonblocking 비동기 코루틴 완전 정복](https://soooprmx.com/python-asycnio-에서-런루프를-기반으로-non-blocking-코드-작성하기/) 
>
> [Task, Future, Coroutine](https://soooprmx.com/asyncio/) 
>
> [Awaitable에 대해](https://soooprmx.com/python-awaitable에-대해/) 

asyncio는 coroutine 방식이다. coroutine을 만들려면 아래와 같이 함수 앞에 async 키워드를 붙이면 된다.

async 함수를 실행하면 함수 내용이 바로 실행되는게 아니라 coroutine이 생성된다.

```python
def sync_func():
    pass

async def async_func():
    pass

print(sync_func()) # None
print(async_func()) # <coroutine object get at 0x10d004120>
```

coroutine은 await를 만나는 순간 실행되기 시작한다.

예를 들어 아래와 같이 실행한다면 첫번째 `async_func()` coroutine을 생성한 뒤 바로 await로 실행하고

이어서 두번째 `async_func()` coroutine을 생성해서 또 바로 await로 실행하는데 이렇게 하면 sync 방식처럼 차례로 실행되어서 총 2초가 걸린다.

```python
import asyncio, time

async def async_func():
    await asyncio.sleep(1)

async def main():
    await async_func()
    await async_func()

start = time.time()
asyncio.run(main())
print(time.time() - start)
```

coroutine 여러개를 async 방식으로 실행하고 싶다면

`asyncio.create_task(async_func())` 로 여러 개의 `async_func()` coroutine을 Task로 감싸서 event loop에 추가한다.

`await t2` 는 실행하지도 않는다는 점에 주목한다. `await t1` 을 실행하는 순간 event loop에 Task로 감싸져서 추가되었던 coroutine들이 동시에 async로 실행되며 1초가 걸리게 된다.

```python
import asyncio, time

async def async_func():
    await asyncio.sleep(1)

async def main():
    # coroutine을 event loop에 task로 추가하고 실행하지는 않음
    t1 = asyncio.create_task(async_func())
    t2 = asyncio.create_task(async_func())
    await t1 # 이 때 실행됨

start = time.time()
asyncio.run(main())
print(time.time() - start)
```

coroutine을 여러개 동시에 돌리는 방법은 다양하다.

```python
# task 함수 사용
## for loop 방식. 그러나 첫번째 await에서 이미 coroutine들이 실행되므로 그 이후의 await은 의미가 없다.
[await task for task in [asyncio.create_task(get(i)) for i in range(2)]]
## await는 1회만 돌려도 된다는 점에 착안해 list의 요소 1개만 뽑아서 await 해준다. 그러나 list가 비었다면 에러가 나겠지.
await [asyncio.create_task(get(i)) for i in range(2)].pop()

# gather 함수 사용
await asyncio.gather(get(1), get(2))
## for loop로 list 만들고 *로 unpacking해서 gather 파라미터로 전달한다.
await asyncio.gather(*[get(i) for i in range(2)])
```

다만 async & await로 coroutine을 호출한다고 해서 무조건 async로 동작하는 것은 아니다. 예를 들어 아래 코드 수행 결과는 2초다.

coroutine이라 해도 blocking 함수를 호출하는 순간 event loop가 차단된다.

```python
import asyncio, time

async def async_func():
    time.sleep(1) # blocking

async def main():
    t1 = asyncio.create_task(async_func())
    t2 = asyncio.create_task(async_func())
    await t1

start = time.time()
asyncio.run(main())
print(time.time() - start)
```

이와 같은 blocking 함수를 non-blocking으로 바꿔서 실행하고 싶다면 아래와 같이 변경한다.

```python
import asyncio, functools, time

def async_func(i: int, s: str):
    print(i, s)
    time.sleep(1)
    return f"{i} {s}"

async def main():
    loop = asyncio.get_event_loop()
    # args, kwargs를 전달해야 하는 경우 functools.partial()을 사용한다.
    # loop.run_in_executor()를 나란히 호출시 2개의 async_func()이 동시에 실행되며 각각의 Future가 리턴된다.
    f1 = loop.run_in_executor(None, functools.partial(async_func, 1, s="1st"))
    f2 = loop.run_in_executor(None, functools.partial(async_func, 2, s="2nd"))
    print(f1, f2)

    # 차례대로 실행하며 async_func() 실행 결과를 리턴한다.
    r1 = await f1
    r2 = await f2
    print(r1, r2)

    # async_func() 실행 결과를 모아서 리턴한다.
    r1 = await asyncio.gather(f1, f2)
    print(r1)
    
start = time.time()
asyncio.run(main())
print(time.time() - start)
```

매번 loop.run_in_executor() 호출하기 번거로우니 decorator를 사용하자.

```python
import asyncio, functools, time

def to_async(f):
    @functools.wraps(f)
    def inner(*args, **kwargs):
        loop = asyncio.get_event_loop()
        return loop.run_in_executor(None, functools.partial(f, *args, **kwargs))
    return inner

@to_async
def test(i):
    print(i)
    time.sleep(1) # blocking function

async def main():
    f1 = test(1)
    f2 = test(2)

start = time.time()
asyncio.run(main())
print(time.time() - start)
```

주의할 점은 `asyncio.run()` 은 호출할 때마다 새로운 event loop를 생성한다. `loop.run_in_executor()` 는 현재의 event loop에 threading을 사용하므로 좀 더 효율적이다.

실제로 아래와 같이 둘의 성능을 비교해보면 `loop.run_in_executor()` 쪽이 좀 더 빠르다.

그래도 native coroutine만큼 효율적이진 않다. 최대한 `native coroutine` >  `loop.run_in_executor()` > `asyncio.run()` 순으로 사용하자.

```python
import asyncio, functools, time

def to_async(f):
    @functools.wraps(f)
    def inner(*args, **kwargs):
        loop = asyncio.get_event_loop()
        return loop.run_in_executor(None, functools.partial(f, *args, **kwargs))
    return inner

async def loop_run_in_executor():
    @to_async
    def test():
        pass
    for _ in range(1000):
        await test()

def asyncio_run():
    async def test():
        pass
    for _ in range(1000):
        asyncio.run(test())

if __name__ == '__main__':
    start1 = time.time()
    asyncio.run(loop_run_in_executor())
    print(f"{time.time() - start1:.2f}") # 0.10

    start2 = time.time()
    asyncio_run()
    print(f"{time.time() - start2:.2f}") # 0.30
```

정리하면 아래와 같다.

```python
if blocking:
    if run a task at a time:
        await loop.run_in_executor
    elif run many tasks concurrently:
        loop.run_in_executor
elif non_blocking:
    if run a task at a time:
        async, await
    elif run many tasks concurrently:
        asyncio.create_task, asyncio.gather
```

blocking인 함수를 `loop.run_in_executor` 로 감싸면 non-blocking이 된다. 그래서 await를 붙여서 한번에 하나씩 non-blocking 작업을 할 수 있다.

다만 `async func()` 은 `func()` 으로 호출시 coroutine만 리턴되고 이후 `await` , `asyncio.create_task` , `asyncio.gather` 를 만나야 비로소 실행이 되는 반면

`loop.run_in_executor` 는 호출시 await가 없으면 바로 `func()` 이 실행된다. 이후 `asyncio.gather` 를 실행할 수는 있지만 이미 수행된 `func()` 리턴값을 모아보는 역할뿐이다.



asyncio로 subprocess를 실행:

> https://docs.python.org/ko/3/library/asyncio-subprocess.html

```python
import asyncio

async def main():
    t1 = await asyncio.create_subprocess_shell(
        "sleep 2; echo 1",
        stdout=asyncio.subprocess.PIPE,
        stderr=asyncio.subprocess.PIPE)
    t2 = await asyncio.create_subprocess_shell(
        "sleep 2; echo 2",
        stdout=asyncio.subprocess.PIPE,
        stderr=asyncio.subprocess.PIPE)
    results = await asyncio.gather(t1.communicate(), t2.communicate())
    print(f'return t1: {t1.returncode}')
    print(f'return t2: {t2.returncode}')
    for stdout, stderr in results:
        print(f'stdout: {stdout.decode() if stdout else ""}')
        print(f'stderr: {stderr.decode() if stderr else ""}')

asyncio.run(main())
```



# functools

## cache

어떤 함수를 동일한 parameter로 호출했을 때, return 값도 동일하다면 매번 다시 연산할 필요는 없다. cache는 함수의 결과를 캐싱해주는 역할을 한다.

다만 기본으로 ttl 기능이 제공되지 않는데 아래와 같이 구현하면 우아하게 처리할 수 있다.

> https://stackoverflow.com/questions/31771286/python-in-memory-cache-with-time-to-live/55900800

```python
from functools import lru_cache
import time

# 예를 들어 seconds = 5라면 1/5 ~ 5/5로 5초가 지나야만 값이 1씩 올라가서 parameter 값이 바뀌게 된다.
def ttl(seconds=3600):
    """ Return the same value withing `seconds` time period """
    return round(time.time() / seconds)

# cache가 아닌 lru_cache를 사용하고 반드시 maxsize를 줘서 메모리 누수가 없도록 한다.
@lru_cache(maxsize=12)
def main(i, ttl=None):
    del ttl # to emphasize we don't use it and to shut pylint up
    time.sleep(1)
    return i

if __name__ == '__main__':
    # 5초간 지속되는 캐싱값을 생성한다.
    for i in range(3):
       print(main(i, ttl=ttl(5)))

    # 캐싱된 값이 즉시 리턴된다.
    for i in range(3):
       print(main(i, ttl=ttl(5)))
```



# Context Manager

> [yield from – 다른 제너레이터에게 위임하기](https://soooprmx.com/yield-다른-제너레이터에게-작업을-위임하기/) 

만약 DB Operation 수행시 Transaction 및 commit/rollback을 수동으로 관리한다면 실수에 의해 로직이 빵꾸날 가능성이 있다. 

Context Manager 패턴을 사용하여 Transaction을 자동화하자. 

```python
from sqlalchemy import declarative_base, create_engine, sessionmaker
from contextlib import contextmanager

Base = declarative_base()
engine = create_engine('...')
Session = sessionmaker(bind=engine)

@contextmanager
def session_scope():
    session = Session()
    try:
        yield session
        session.commit()
    except:
        session.rollback()
        raise
    finally:
        session.close()

with session_scope() as session:
    session.query(...)
```

위와 같이 실행하면 with문 내에서 session을 받아오는 순간 session_scope 함수의 yield까지만 실행되고

그 이후부터는 with문 로직을 모두 실행한 뒤에 다시 session_scope 함수의 yield 이후가 실행된다.

즉 with문 로직이 정상이면 commit, 예외 발생시 rollback을 한 뒤, 마지막으로 session.close()가 실행된다.



# direnv, virtualenv

virtualenv로 python 환경을 독립적으로 구성할 수 있다. 

예를 들어 project 2개가 있는데 서로 다른 django 버전을 사용한다면? 혹은 system wide로는 필요없는 라이브러리들인데도 project 때문에 덕지덕지 설치해야 한다면?

virtualenv를 사용하여 깨끗한 python 환경을 만들고 project용으로 구성하면 system wide는 물론 project 간에도 서로에게 영향을 미치지 않고 독립된 무균실에서 개발 가능하다. 

그러나 virtualenv 사용시에는 매번 activate/deactivate를 실행해야 하는 번거로움이 있으니 direnv로 특정 dir에 들어갈 때마다 virtualenv가 자동 실행되도록 하자. 아래와 같이 설치한다. 

```sh
sudo apt install -y direnv
pip install virtualenv

# ~/.zshrc 에 다음 사항 추가
eval "$(direnv hook zsh)"

# ~/.bashrc 에 다음 사항 추가
eval "$(direnv hook bash)"
```

이후 프로젝트 경로에 들어가서 아래와 같이 `.envrc` 파일을 만들고 

```sh
layout_python3
```

direnv allow한다. 첫 진입시에는 환경을 초기 구성하느라 조금 시간이 걸릴 수 있다. 

```sh
direnv allow
```

이후부터 프로젝트 경로에 진입하면 pip, python 등을 system bin 대신 .direnv 경로에 있는 것으로 사용하는 것을 알 수 있다. 

```sh
> ~ where pip python
/usr/bin/pip
/usr/bin/python

> ~ cd ~/repo/me/test/fastapi 
direnv: loading ~/repo/me/test/fastapi/.envrc
direnv: export +VIRTUAL_ENV ~PATH

> ~/repo/me/test/fastapi where pip python
/home/repo/me/test/fastapi/.direnv/python-3.10.2/bin/pip
/usr/bin/pip
/home/repo/me/test/fastapi/.direnv/python-3.10.2/bin/python
/usr/bin/python
```

또한 system-wide로는 설치된 python package라도 프로젝트 경로에서는 import도 안되는 등 독립적인 환경인 것을 알 수 있다.

```sh
> ~ python
Python 3.10.2 (default, Jun 30 2021, 09:17:59)
>>> import fastapi
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ModuleNotFoundError: No module named 'fastapi'
>>>

> ~ cd ~/repo/me/test/fastapi
direnv: loading ~/repo/me/test/fastapi/.envrc
direnv: export +VIRTUAL_ENV ~PATH

> ~/repo/me/test/fastapi python
Python 3.10.2 (default, Jun 30 2021, 09:17:59)
>>> import fastapi
>>> 
```

만약 Bash script 내에서 direnv를 적용해야 한다면 아래와 같이 실행해준다.

```sh
cd repo/me/test/python
direnv allow . && eval "$(direnv export bash)"
python ...
```



# pyenv

pyenv는 다양한 버전의 python을 입맛대로 골라 사용할 수 있게 해준다. 

예를 들어 여러 시스템에서 환경 동기화를 위해 동일 버전의 python을 사용해야 할 수 있다. 혹은 반대로 한 시스템에서 여러 버전의 python을 사용해야 할 수도 있다. 아래와 같이 설치한다.

> https://github.com/pyenv/pyenv-installer

```sh
# Mac
brew install pyenv pyenv-virtualenv

# Ubuntu
curl https://pyenv.run | bash
```

.zshrc, .bashrc에 아래와 같이 추가해준다.

```sh
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

pyenv install 시 빌드에 문제가 생기지 않도록 아래 문서를 참조해 Prerequisites부터 준비한다.

> https://github.com/pyenv/pyenv/wiki#suggested-build-environment
>
> https://github.com/pyenv/pyenv/wiki/Common-build-problems

```sh
# Mac
brew install openssl readline sqlite3 xz zlib

# Ubuntu
sudo apt-get update
sudo apt-get install make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev libedit-dev

# Termux : 얘는 아래 패키지 깔아도 안됨
apt install -y build-essential libbz2 wget curl llvm xz-utils tk libxml2 libffi liblzma libedit
```

원하는 경로로 이동하여 원하는 python 버전을 설치한다. 소스를 받아 빌드하므로 시간이 오래 걸린다.

```sh
pyenv install 3.10.2
```

이후 .envrc에 아래와 같이 정의한다.

```sh
if command -v pyenv &> /dev/null; then
    use_python() {
      local python_root=$HOME/.pyenv/versions/$1
      load_prefix "$python_root"
      layout_python "$python_root/bin/python"
    }

    use python 3.10.2
else
    layout_python3
fi
```



# pip

아래와 같이 여러 패키지를 설치하는 경우 혹은 버전까지 붙여서 패키지를 관리하는 경우

```sh
pip install package1 package2 package3 ...
```

보통 `requirements.txt` 파일에 아래와 같이 패키지를 넣고

```sh
package1
package2
package3
...
```

아래와 같이 한방에 설치한다.

```sh
pip install -r requirements.txt
```



# Simple Web Server

```sh
python3 -m http.server 9000
```



# JupyterLab

```sh
# install
pip install jupyterlab

# init
jupyter notebook password
jupyter lab --generate-config

# 외부 접속 설정
# c.ServerApp.ip = '*'
# c.ServerApp.port = 8888
# c.ServerApp.allow_origin = '*'
# c.ServerApp.allow_remote_access = True

# SSL 설정
# c.ServerApp.certfile = '/data/data/com.termux/files/home/repo/acme-tiny/letsencrypt.crt'
# c.ServerApp.keyfile = '/data/data/com.termux/files/home/repo/acme-tiny/letsencrypt.key'

jupyter server stop # 반드시 이렇게 종료해주자. kill로 강제 종료하면 checkpoint error 발생 가능

# extensions
jupyterlab-go-to-definition
pip install jupyterlab-lsp
pip install jupyterlab-drawio
```



# FastAPI

> [FastAPI](https://fastapi.tiangolo.com/ko/features/) 
>
> [Background Tasks](https://fastapi.tiangolo.com/tutorial/background-tasks/) 
>
> [Uvicorn](https://github.com/encode/uvicorn) 

```sh
pip install fastapi uvicorn[standard] python-multipart
```

`uvicorn` 은 pure python으로 최소한의 dependency만 설치한다.

`uvicorn[standard]` 는 Cython 기반이며 `uvloop` 를 사용하므로 가능하면 `uvicorn[standard]` 를 사용한다.



## Deploy

ASGI 서버는 몇 가지가 있는데, FastAPI 공식 문서를 보면 그 중 uvicorn을 표준으로 사용한다. 

> `--reload` 사용시 `--workers` 는 무시되므로 개발할때만 사용하자. 또 `log level` default가 `info` 라 성능에 영향을 주니 error로 변경해주자.

```sh
uvicorn main:app --workers 4 --host 0.0.0.0 --port 8080 --log-level error
```

uvicorn 앞단에 gunicorn을 붙이는 것도 가능하다. https://www.uvicorn.org/deployment/ 를 참조하여 아래와 같이 실행해준다.

[Gunicorn Docs](https://docs.gunicorn.org/en/stable/settings.html#worker-processes), [Better performance by optimizing Gunicorn config](https://medium.com/building-the-system/gunicorn-3-means-of-concurrency-efbb547674b7) 를 보면 `number of workers = (cpu cores * 2) + 1` 와 같이 계산한다.

```sh
gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker -b 0.0.0.0:8080
```

혹은 gunicorn or uvicorn 앞단에 nginx를 붙일 수도 있다. nginx에서 바로 FastAPI의 ASGI 서버 역할을 해줄 수도 있다.

> [FastAPI - NGINX Unit](https://unit.nginx.org/howto/fastapi/) 



## on_event

> [Events: startup - shutdown](https://fastapi.tiangolo.com/advanced/events/) 

아래는 startup 시 subprocess를 실행하는 예제이다.

Pub/Sub의 Subscriber 처럼 API를 호출받는게 아니라 Pull 방식으로 데이터를 끌어오는 아이라면 background로 subprocess를 돌려놓는 것이 좋겠다.

```python
import subprocess

from fastapi import FastAPI

PREFIX = "/fastapi"

app = FastAPI(openapi_url=f"{PREFIX}/c/openapi.json",
              docs_url=f"{PREFIX}/c/docs", redoc_url=None)

bg_jobs = []


@app.on_event("startup")
def startup():
    bg_jobs.append(subprocess.Popen(["python3", "-m", "app.pubsub.pub"]))
    bg_jobs.append(subprocess.Popen(["python3", "-m", "app.pubsub.sub"]))


@app.on_event("shutdown")
def shutdown():
    for job in bg_jobs:
        job.kill()
```



# httpie

> https://httpie.io/



# pyautogui

리눅스에서 `You must install tkinter on Linux to use MouseInfo. Run the following: sudo apt-get install python3-tk python3-dev` 에러 발생시

> https://stackoverflow.com/questions/26357567/cannot-import-tkinter-after-installing-python-3-with-pyenv

```sh
sudo apt-get install tk-dev

pyenv install 3.10.2

# test
python3 -m tkinter
```



# ntfy

터미널 명령어로 OS notify를 보내준다.

> https://github.com/dschep/ntfy

```sh
pip install nyft

# 첫번째 커맨드 실행이 성공하면 success를, 실패하면 fail을 notify 해준다.
sleep 3 && ntfy send success || ntfy send fail
```



# AsyncSSH

> https://asyncssh.readthedocs.io/en/latest/



# paramiko

```sh
pip install paramiko
```

```python
import paramiko

def connect(host, port, username, pk_path):
    client = paramiko.SSHClient()
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    client.connect(hostname=host, port=port, username=username,
                   pkey=paramiko.RSAKey.from_private_key_file(pk_path))
    return client

def main():
    client = connect(host="test.com", port=22,
                     username="user", pk_path="/test/.ssh/id_rsa")
    cmd = f"hostname"
    _, stdout, stderr = client.exec_command(cmd)
    exit_status = stdout.channel.recv_exit_status() # blocking
    if exit_status == 0:
        output = stdout.readlines()
        print("OK", output)
    else:
        print("Error", exit_status, stderr)
    client.close()

if __name__ == '__main__':
    main()
```



# Faker

테스트용 가짜 데이터를 만들어주는 패키지

> [Faker](https://faker.readthedocs.io/en/master/) 



# Selenium

웹 테스트 자동화 프레임워크

> [Selenium](https://www.selenium.dev/) 
>
> [Selenium IDE](https://www.selenium.dev/selenium-ide/) 



**Install Packages** 

```sh
pip install selenium webdriver_manager
```



**Install ChromeDriver** 

직접 [ChromeDriver](https://sites.google.com/chromium.org/driver/) 사이트에 가서 바이너리를 다운받고 경로 설정을 해 줄 수도 있지만

아래와 같이 `ChromeDriverManager` 를 사용하면 내 Chrome 버전에 맞는 Driver가 없으면 자동으로 다운받고, 다음번 실행시에는 캐싱된 Driver 파일로 구동된다.

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

chrome = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=webdriver.ChromeOptions())
```



**Sample Code** 

- asyncio 버전. async라 sync보단 빠르지만 driver마다 별도의 chrome process가 실행되기 때문에 cpu bound 비중이 커서 싱글코어 제약의 영향을 받는다.

```python
# pylint: disable=missing-module-docstring
# pylint: disable=missing-class-docstring
# pylint: disable=missing-function-docstring

# dependencies
# 1. pip install selenium webdriver_manager
# 2. Add this in Windows Task Scheduler: C:\Users\user\AppData\Local\Programs\Python\Python310\python.exe "C:\bin\wake-up.py"

import asyncio

from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from webdriver_manager.chrome import ChromeDriverManager

# driver를 함수 내 로컬 변수로 쓰면 함수 종료시 driver가 gc되므로 reference count를 유지하기 위해 drivers에 넣어둠
drivers = []

async def open_driver():
    driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=webdriver.ChromeOptions())
    drivers.append(driver)
    return driver

async def signin_site247(driver):
    driver.get("https://site247")
    driver.switch_to.frame("loginIframe")
    driver.find_element(by=By.NAME, value="username").send_keys("id")
    driver.find_element(by=By.NAME, value="password").send_keys("passwd")
    driver.find_element(by=By.NAME, value="signinform").submit()
    await asyncio.sleep(5) # wait for signing in

async def signin_whatap(driver):
    driver.get("http://whatap")
    driver.find_element(by=By.ID, value="id_email").send_keys("id")
    driver.find_element(by=By.ID, value="id_password").send_keys("passwd")
    driver.find_element(by=By.ID, value="btn_login").click()
    await asyncio.sleep(5) # wait for signing in

async def expand_site247(driver):
    try:
        WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.CLASS_NAME, "icon-expand")))
    finally:
        driver.find_element(by=By.CLASS_NAME, value="icon-expand").click()


# PC 1 Screens
# [            driver1_1            ]
# [driver1_2] [driver1_3] [driver1_4]
# [driver1_5]

# 디스플레이 해상도가 3840*2160 이고 1.5 배율일때
driver1_max_width, driver1_max_height = 3840/1.5, 2160/1.5

async def run1_1():
    driver = await open_driver()
    # new driver를 열 때마다 별도의 sandbox 브라우저가 뜨기 때문에 로그인이나 쿠키가 공유되지 않는다. 로그인이 필요하면 매번 새로 해야함
    await signin_site247(driver)
    driver.get("https://1_1")
    driver.set_window_position(0, -driver1_max_height) # send browser to upper monitor
    # fullscreen window로 띄우는 경우 갑자기 최초 기본 창 크기로 resize 되는 경우가 있어 fullscreen 대신 window size를 거의 full로 채우는 방식으로 적용
    driver.set_window_size(driver1_max_width, driver1_max_height-50) # height에서 작업표시줄 길이 빼줌
    await expand_site247(driver)

async def run1_2():
    driver = await open_driver()
    await signin_site247(driver)
    driver.get("https://1_2")
    driver.set_window_position(0, 0)
    driver.set_window_size(driver1_max_width/3, driver1_max_height/2-25)
    await expand_site247(driver)

async def run1_3():
    driver = await open_driver()
    await signin_site247(driver)
    driver.get("https://1_3")
    driver.set_window_position(driver1_max_width/3, 0)
    driver.set_window_size(driver1_max_width/3, driver1_max_height-50)
    await expand_site247(driver)

async def run1_4():
    driver = await open_driver()
    await signin_site247(driver)
    driver.get("https://1_4")
    driver.set_window_position(driver1_max_width/3*2, 0)
    driver.set_window_size(driver1_max_width/3, driver1_max_height-50)
    await expand_site247(driver)

async def run1_5():
    driver = await open_driver()
    await signin_site247(driver)
    driver.get("https://1_5")
    driver.set_window_position(0, driver1_max_height/2)
    driver.set_window_size(driver1_max_width/3, driver1_max_height/2-25)
    await expand_site247(driver)

async def main1():
    await asyncio.sleep(30) # wait for os init
    await asyncio.gather(run1_1(), run1_2(), run1_3(), run1_4(), run1_5())


# PC 2 Screens
# [driver2_1]
# [driver2_2]

driver2_max_width, driver2_max_height = 3840/1.5, 2160/1.5

async def run2_1():
    driver = await open_driver()
    await signin_site247(driver)
    driver.get("https://2_1")
    driver.set_window_position(0, 0)
    driver.set_window_size(driver2_max_width, driver2_max_height-50)
    await expand_site247(driver)

async def run2_2():
    driver = await open_driver()
    await signin_whatap(driver)
    driver.get("http://2_2")
    driver.set_window_position(0, -driver2_max_height)
    driver.set_window_size(driver2_max_width, driver2_max_height-50)

async def main2():
    await asyncio.sleep(60) # wait for os init
    await asyncio.gather(run2_1(), run2_2())

if __name__ == '__main__':
    asyncio.run(main1())
    asyncio.run(main2())
```



- multiprocessing 버전. 멀티코어를 사용하므로 이쪽이 훨씬 빠르다. 다만 여러 프로세스가 동시에 open_driver()를 실행할 때 Windows 환경에서 (다른 환경에는 아직 발견하지 못함)
    `selenium.common.exceptions.WebDriverException: Message: 'chromedriver.exe' executable may have wrong permissions. Please see https://chromedriver.chrominum.org/home` 에러가 발생하는 것이 발견된다. 먼저 chrome driver 초기화를 완전히 끝낸 후에 이후 로직을 진행해야겠다.

```python
# pylint: disable=missing-module-docstring
# pylint: disable=missing-class-docstring
# pylint: disable=missing-function-docstring

# dependencies
# 1. pip install selenium webdriver_manager
# 2. Add this in Windows Task Scheduler: C:\Users\user\AppData\Local\Programs\Python\Python310\python.exe "C:\bin\wake-up.py"

import time
from multiprocessing import Process, Pool

from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
from webdriver_manager.chrome import ChromeDriverManager

# driver를 함수 내 로컬 변수로 쓰면 함수 종료시 driver가 gc되므로 reference count를 유지하기 위해 drivers에 넣어둠
drivers = []

def open_driver():
    driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=webdriver.ChromeOptions())
    drivers.append(driver)
    return driver

def signin_site247(driver):
    driver.get("https://site247")
    driver.switch_to.frame("loginIframe")
    driver.find_element(by=By.NAME, value="username").send_keys("id")
    driver.find_element(by=By.NAME, value="password").send_keys("pass")
    driver.find_element(by=By.NAME, value="signinform").submit()
    time.sleep(5) # wait for signing in

def signin_whatap(driver):
    driver.get("http://whatap")
    driver.find_element(by=By.ID, value="id_email").send_keys("id")
    driver.find_element(by=By.ID, value="id_password").send_keys("pass")
    driver.find_element(by=By.ID, value="btn_login").click()
    time.sleep(5) # wait for signing in

def expand_site247(driver):
    try:
        WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.CLASS_NAME, "icon-expand")))
    finally:
        driver.find_element(by=By.CLASS_NAME, value="icon-expand").click()


# PC 1 Screens
# [         run1_1         ]
# [run1_2] [run1_3] [run1_4]
# [run1_5]

# 디스플레이 해상도가 3840*2160 이고 1.5 배율일때
driver1_max_width, driver1_max_height = 3840/1.5, 2160/1.5

def run1_1():
    driver = open_driver()
    # new driver를 열 때마다 별도의 sandbox 브라우저가 뜨기 때문에 로그인이나 쿠키가 공유되지 않는다. 로그인이 필요하면 매번 새로 해야함
    signin_site247(driver)
    driver.get("https://1_1")
    driver.set_window_position(0, -driver1_max_height) # send browser to upper monitor
    # fullscreen window로 띄우는 경우 갑자기 최초 기본 창 크기로 resize 되는 경우가 있어 fullscreen 대신 window size를 거의 full로 채우는 방식으로 적용
    driver.set_window_size(driver1_max_width, driver1_max_height-50) # height에서 작업표시줄 길이 빼줌
    expand_site247(driver)

def run1_2():
    driver = open_driver()
    signin_site247(driver)
    driver.get("https://1_2")
    driver.set_window_position(0, 0)
    driver.set_window_size(driver1_max_width/3, driver1_max_height/2-25)
    expand_site247(driver)

def run1_3():
    driver = open_driver()
    signin_site247(driver)
    driver.get("https://1_3")
    driver.set_window_position(driver1_max_width/3, 0)
    driver.set_window_size(driver1_max_width/3, driver1_max_height-50)
    expand_site247(driver)

def run1_4():
    driver = open_driver()
    signin_site247(driver)
    driver.get("https://1_4")
    driver.set_window_position(driver1_max_width/3*2, 0)
    driver.set_window_size(driver1_max_width/3, driver1_max_height-50)
    expand_site247(driver)

def run1_5():
    driver = open_driver()
    signin_site247(driver)
    driver.get("https://1_5")
    driver.set_window_position(0, driver1_max_height/2)
    driver.set_window_size(driver1_max_width/3, driver1_max_height/2-25)
    expand_site247(driver)

def main1():
    time.sleep(30) # wait for os init
    with Pool(processes=5) as pool:
        for func in [run1_1, run1_2, run1_3, run1_4, run1_5]:
            pool.apply_async(func=func)
        pool.close()
        pool.join()


# PC 2 Screens
# [run2_1]
# [run2_2]

driver2_max_width, driver2_max_height = 3840/1.5, 2160/1.5

def run2_1():
    driver = open_driver()
    signin_site247(driver)
    driver.get("https://2_1")
    driver.set_window_position(0, 0)
    driver.set_window_size(driver2_max_width, driver2_max_height-50)
    expand_site247(driver)

def run2_2():
    driver = open_driver()
    signin_whatap(driver)
    driver.get("https://2_2")
    driver.set_window_position(0, -driver2_max_height)
    driver.set_window_size(driver2_max_width, driver2_max_height-50)

def main2():
    time.sleep(60) # wait for os init
    for func in [run2_1, run2_2]:
        Process(target=func).start()

if __name__ == '__main__':
    main1()
    main2()
```



# Issues

## 도메인 변경 감지

```python
import os
import datetime

def get_ips_by_dns_lookup(host, port=80):
    import socket
    try:
        return sorted(list(map(lambda x: x[4][0], socket.getaddrinfo(f"{host}.", port, type=socket.SOCK_STREAM))))
    except:
        return ["can't lookup"]

hosts = ["test.com", "www.test.com"]
# find home dir. ex: /home/test
dir = os.path.join(os.path.expanduser("~"), "restart_when_ip_changed")

for host in hosts:
    file = f"{dir}/{host}.txt"
    os.system(f"touch {file}")

    with open(file, "r") as file_r:
        old_ip = file_r.read()
        new_ip = str(get_ips_by_dns_lookup(host))

        if old_ip != new_ip:
            print(f"{datetime.datetime.now() + datetime.timedelta(hours=9)}(KST) {host}: {old_ip} != {new_ip}")
            os.system("sudo docker restart test")

            with open(file, "w") as file_w:
                file_w.write(new_ip)
        else:
            print(f"{datetime.datetime.now() + datetime.timedelta(hours=9)}(KST) {host}: {old_ip} == {new_ip}")
```

```sh
# run script every 1 minute
*/1 * * * * python3 /home/test/restart_when_ip_changed.py >> /home/test/restart_when_ip_changed/log

# rm log every monday 01:00
00 1 * * 1 echo > /home/test/restart_when_ip_changed/log

# restart test everyday 01:00
00 01 * * * sudo docker restart test >> /home/test/restart_when_ip_changed/log
```

