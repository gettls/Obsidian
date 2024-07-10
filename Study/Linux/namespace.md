
```python
// a.py
def hello():
	print("hello from A!")

// b.py
def hello():
	print("hello from B!")
```

`a.py`,`b.py`에 서로 같은 시그니처의 함수가 있다고 가정해보자

```python
from a import hello
from b import hello

hello()
```

만약 위와 같이 import를 해서 사용하고자 한다면, 같은 시그니처를 가지고 있는 두 함수는 서로 충돌이 일어날 것이다.

```python
import a
import b

a.hello()
b.hello()
```

그래서 우리는 위와 같이 패키지를 명시적으로 선언해줌으로써 이러한 충돌을 해결한다.

Linux에서도 이러한 기능을 제공하는게 있는데, 그게 바로 `namespace`이다.

### namespace

파일/네트워크/권한 같은 것들을 격리하고자 하는 목적으로 존재한다.

예를 들어서, 프로세스를 격리해주는 `namespace` 는 `PID namespace`이고, 파일/폴더에 대한 권한을 격리하는 `namespace`는 `user namespace` 이다.
