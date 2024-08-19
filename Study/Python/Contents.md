# Day 1

**가상환경**
- 프로젝트에서 사용되는 의존성을 관리해준다. 가상환경을 사용함으로써 개발자는 의존성의 버저닝을 관리할 필요가 없어진다.

**keyword**
- 미리 정의되어 있는 예약어를 의미한다. 이 키워드들은 다른 용도로 사용할 수 없다.
```python
import keyword  
  
print(keyword.kwlist)

>>> 
['False', 'None', 'True', 'and', 'as', 'assert', 'async', 'await', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return', 'try', 'while', 'with', 'yield']
```

**문자열**
- 문자열 서식화
```python
name = "Alice"  
age = 16  
print(f"Name:{name}, age:{age}") # f-strings - Python 3.6+
print("Name:{}, age:{}".format(name, age)) # format()
print("Name:%s, age:%d" % (name, age)) # % 연산자
```

**숫자 변수**
- Integer
	- 10진수 : 일반적인 정수
	- 2진수 : `0b` or `0B`로 시작
	- 8진수 : `0o` or `0O`로 시작
	- 16진수 : `0x` or `0X`로 시작
- Float
	- 실수형
- Complex
	- 복소수형
```python
c = 3 + 4j
print(c.real) # 3.0
print(c.imag) # 4.0
```
- 형 변환
```python
# 실수형을 정수형으로 변환
x = 3.14
print(int(x))  # 출력: 3

# 정수형을 실수형으로 변환
y = 10
print(float(y))  # 출력: 10.0

# 정수형을 복소수형으로 변환
z = 7
print(complex(z))  # 출력: (7+0j)

# 실수형을 복소수형으로 변환
w = 1.5
print(complex(w))  # 출력: (1.5+0j)
```


**문서 문자열(doc string)**
```python
def add(a, b):  
    """  
    두 수를 더한 값을 출력한다.  
    a (int): 첫 번째 숫자  
    b (int): 두 번째 숫자  
    """    return a+b  
print(add.__doc__)

class Person:  
    """  
    Person 클래스 선언입니다.  
    """    def __init__(self, name, age):  
        """  
        Person의 생성자입니다.  
        매개변수 :        name(str): 이름  
        age(integer): 나이  
        """        self.name = name  
        self.age = age  
    def walk(self):  
        """  
        사람이 걷는 메서드입니다.  
        """        print("working...")  
  
print(Person.__doc__)  
print(Person.__init__.__doc__)  
print(Person.walk.__doc__)
```

- 문서 문자열 활용
```terminal
pydoc -w <filename>
```
- `pydoc` 을 사용하면 주석을 가지고 html 파일을 만들어줌


**기본(default) 매개변수**
```python
def append_to_list(value, my_list=[]):  
    my_list.append(value)  
    return my_list  
  
print(append_to_list(1))  # 출력: [1]  
print(append_to_list(2))  # 출력: [1, 2]  
print(append_to_list(3))  # 출력: [1, 2, 3]
```
- dic, array 과 같은 객체는 my_list는 함수가 호출될 때마다 공유된다. 따라서 의도치 않은 결과를 나을 수 있다.
```python
def append_to_list(value, my_list=None):  
    if my_list is None:  
        my_list = []  
    my_list.append(value)  
    return my_list
```
- 위와 같은 식으로 함수 내부에서 새로운 리스트를 생성하는 식으로 변경해야 한다.

**키워드 인수(keyword argument)**
- `keywork argument` 라는 것은, 메서드를 호출할 때 argument의 `key`, `value`값을 정의해서 호출한다는 것이다.
```python
def describe_person(name ,age):  
    print(f"name: {name} age:{age}")  
  
describe_person(name="Kim", age=30)
describe_person(name="Kim", city="Seoul") # TypeError 
```
- 메서드 시그니처에 정의된 argument와 메서드를 호출할 때 인자로 보낸 argument의 `key`값이 다르면 `TypeError` 가 발생한다.

- `**kwargs & *args`
- `keyword`인수를 배열로 보낼 수도 있다. 이를 `위치인수`라고 한다.
```python
def describe_person(name , **kwargs):  
    for key, value in kwargs.items():  
        print(f"{key} {value}")  
  
  
describe_person(name="kim", city="seoul", married="true")
---
def describe_person(*args, name, **kwargs):  
    for key, value in kwargs.items():  
        print(f"{key} {value}")  
    for arg in args:  
        print(arg)  
  
  
describe_person("arg1", "arg2", name="kim",city="seoul", married="true")
```
- `*args`는 `*kwargs`보다 앞에 와야한다.

- `키워드 전용 인수(*)`
```python
def describe_person(*, name, city):  
    print(f"{name}, {city}")  
  
describe_person("kim", "seoul") # TypeError
```
- `*`의 뒤에 키워드 전용 인수들을 선언하면, 해당 argument들은 키워드 인수로만 호출을 해야한다. -> 안그러면 `TypeError` 발생

# Day 2

**class 변수**
- 클래스 변수 : 클래스 자체에 속하고, 모든 인스턴스가 공유
- 인스턴스 변수 : 개별 인스턴스에 속하며, 각 인스턴스마다 고유한 값을 가짐

**getattr/setattr/delattr 함수**
```python
class Car:  
    brand = "KIA"  
print(getattr(Car, "brand")) # KIA  
setattr(Car, "brand", "NISSAN")  
print(getattr(Car, "brand")) # NISSAN  
delattr(Car, "brand")  
print(getattr(Car, "brand")) # AttributeError
```

**__dict__**
```python
class Car:  
    brand = "KIA"  
  
print(Car.__dict__)
---
{'__module__': '__main__', 'brand': 'KIA', '__dict__': <attribute '__dict__' of 'Car' objects>, '__weakref__': <attribute '__weakref__' of 'Car' objects>, '__doc__': None}
```
객체의 모든 속성과 그 값을 포함하는 딕셔너리를 반환

**메서드**
- `인스턴스 메서드`
```python
class Car:  
    def __init__(self, brand, color):  
        self.brand = brand  
        self.color = color  
  
    def introduce(self):  
        print(f"this car is made by {self.brand} and color is {self.color}")  
  
car = Car("KIA", "red")  
car.introduce()
```
- 인스턴스 메서드는 클래스의 인스턴스에 대해 동작하고, 첫 번째 매개변수로 항상 `self`를 받는다. `self`는 해당 메서드가 호출된 인스턴스를 참조한다.

- `클래스 메서드`
```python
class Car:  
    num = 0 # class 변수 
  
    def __init__(self, brand, color):  
        self.brand = brand  
        self.color = color  
        Car.num += 1  
  
    @classmethod  
    def get_totalSales(cls):  
        print(f"차의 총 대수는 {cls.num} 입니다.")  
  
car1 = Car("KIA", "red")  
car2 = Car("TOYOTA", "blue")  
  
car1.get_totalSales()
```
- 클래스 메서드는 클래스 자체를 대상으로 동작하고, 첫 번째 매개변수로 `cls`를 받는다. `@classmethod` 데코레이터로 정의된다.

- `정적 메서드`
```python
class Cafe:  
  
    def __init__(self, name, location):  
        self.name = name  
        self.location = location  
  
    @staticmethod  
    def describe():  
        print(f"이 곳은 카페입니다.")  
  
  
cafe1 = Cafe("빽다방", "서울")  
cafe2 = Cafe("메가커피", "서울")  
  
Cafe.describe()
```
- 정적 메서드는 클래스나 인스턴스와 관계없이 독립적으로 동작한다. 첫 번째 매개변수로 `self`, `cls` 같은 것을 받지 않고, `@staticmethod` 데코레이터로 정의된다.

**함수(function)**
- 함수란, 클래스나 인스턴스와 연관되지 않으며 독립적으로 동작한다. 그러나, 일반적으로는 클래스 내부에서는 메서드만 정의하고, 함수는 클래스 외부에서 정의를 한다.
- 클래스 내부에 정의된 함수는 클래스나 인스턴스의 맥락에서 동작하지 않기 떄문에 일반적으로 클래스 외부에 두는 것이 좋다.
```python
class Clazz:
	def instance_method(self):
		return "인스턴스 메서드"
	def function():
		return "함수"

clazz = Clazz()

# 클래스 내부에 정의된 함수는 호출 불가
try:
    print(clazz.function())
except TypeError as e:
    print(e)  # TypeError: regular_function() takes 0 positional arguments but 1 was given 출력
```

**프라이빗 변수**
```python
class Person:  
    def __init__(self, name):  
        self.__name = name  
    def get_name(self):  
        return self.__name  
  
person = Person("홍길동")  
print(person.get_name()) 
print(person._Person__name) # 이름장식(name mangling)
try:  
    print(person.__name)  
except AttributeError as e:  
    print(e) # 'Person' object has no attribute '__name'
```
- private variable은 `__`을 통해 선언을 한다. 이렇게 선언된 변수는 외부에서 직접 접근할 수 없다.
- 이름 장식(name mangling)을 통해 접근할 수는 있지만, 권장되는 방식은 아니다.

**스폐셜 메서드**
- `__init__` : 초기화 메서드
- `__str__` : 인스턴스 출력 시 사람이 읽기 쉬운 문자열 형태로 출력
- `__repr__` : 객체의 "공식적인" 문자열 표현, 주로 디버깅 시 사용. repr() 함수나 인터프리터에서 객체를 출력할 때 호출 됨
```python
class Person:
	def __repr__(self):
		return f"Person(name={self.name}, age={self.age})"

person = Person("홍길동", 30)
print(repr(person)) # Person(name=홍길동, age=30) 출력
```
- `__len__` : 길이 반환
```python
class CustomList:
    def __init__(self, items):
        self.items = items

    def __len__(self):
        return len(self.items)

custom_list = CustomList([1, 2, 3, 4, 5])
print(len(custom_list))  # 5 출력
```
- `__getitem__(self,key)` : 객체가 인덱싱될 때 호출되어 해당 키에 해당하는 값을 반환함
```python
class Person:  
    def __init__(self, name):  
        self.name = name  
  
    def __getitem__(self, item):  
        return item * 10  
  
p = Person("홍길동")
print(p[2]) # 여기서 말하는 인덱싱이란, 배열처럼 호출할 때를 의미하는 듯함
```
- `__setitem__(self, key, value)` : 객체의 특정 키에 값을 설정할 때 호출됨
```python
class Person:  
    def __init__(self, name, item):  
        self.name = name  
        self.item = item  
  
    def __getitem__(self, item):  
        return self.item  
  
    def __setitem__(self, key, value):  
        self.item = key + value  
  
p = Person("홍길동", 0)  
p[3] = 6  
print(p[3]) # 9 출력 
```
- `__delitem__(self, key)` : 객체의 특정 키에 해당하는 값을 삭제할 때 호출
```python
class Person:  
    def __init__(self, name, item):  
        self.name = name  
        self.item = item  
  
    def __getitem__(self, item):  
        return self.item  
  
    def __delitem__(self, key):  
        self.item = key
  
p = Person("홍길동", 0)  
  
del p[3]
print(p.item) # 3 출력
```
- `__call__(self, ...)`  : 객체가 함수처럼 호출될 때 실행
```python
class Person:  
    def __init__(self, name, item):  
        self.name = name  
        self.item = item  
  
    def __call__(self, num):  
        return self.item + num + 10000  
  
p = Person("홍길동", 0)  
print(p(5)) # 100005 출력 (0 + 5 + 10000)
```
- `__eq__` : == 연산자를 사용해 동등성을 비교할 때 사용
- `__hash__` : 객체의 해시 값을 반환 함. `set`, `dict`와 같은 해시 기반 컬렉션에서 객체를 키로 사용할 때 필요

**Enum**
```python
from enum import Enum, auto  
  
class Color(Enum):  
    RED = auto() # 1
    BLUE = auto() # 2
  
print(Color.BLUE.value) # 2  
print(Color.RED.value) # 1
print(Color.RED.name) # RED

print(Color.__members__) #{'RED': <Color.RED: 1>, 'BLUE': <Color.BLUE: 2>}

```
- auto()는 값을 자동으로 할당해준다.
- `__members__` 는 모든 멤버를 사전 형태로 반환한다.

**property 함수**
```python
class Circle:  
    def __init__(self, radius):  
        self._radius = radius  
  
    def get_radius(self):  
        return self._radius  
  
    def set_radius(self, value):  
        if value < 0:  
            raise ValueError("반지름은 0보다 커야 합니다.")  
        self._radius = value  
  
    def del_radius(self):  
        del self._radius  
  
    radius = property(get_radius, set_radius, del_radius, "원의 반지름을 나타내는 속성")  
  
# 사용 예시  
circle = Circle(5)  
print(circle.radius)  # 5  
  
circle.radius = 10  
print(circle.radius)  # 10  
  
try:  
    circle.radius = -3  # ValueError 발생  
except ValueError as e:  
    print(e)  
  
del circle.radius  
try:  
    print(circle.radius)  # AttributeError 발생  
except AttributeError as e:  
    print(e)
```
- `property(fget, fset, fdel, doc)`
	- `fget`: 속성 값을 읽을 때 호출(getter)
	- `fset`: 속성 값을 설정할 때 호출(setter)
	- `fdel`: 속성 값을 삭제할 때 호출(deleter)
	- `doc`: 속성에 대한 문서화 문자열

- `@property`
메서드를 속성처럼 사용할 수 있게 됨. `getter` 메서드를 정의할 때 사용함
```python
class Person:  
    def __init__(self, name):  
        self._name = name  
  
    @property  
    def name(self):  
        return self._name  
        
p = Person("홍길동")  
print(p.name) # 홍길동 출력
```

- `응용`
```python
class Person:  
    def __init__(self, name):  
        self._name = name  
  
    @property  
    def name(self):  
        return self._name  
  
    @name.setter  
    def name(self, name):  
        self._name = name  
  
p = Person("홍길동")  
print(p.name)  # 홍길동 출력
  
p.name = "아무개"  
print(p.name) # 아무개 출력
```

# Day 3

**__slots__**
- `__slots` 는 클래스 속성을 고정하여 메모리 사용을 최적화하고, 성능을 향상시키는데 사용된다.
- 파이썬 객체는 `__dict__` 라는 딕셔너리를 사용해 속성을 저장하지만, `__slots__`를 사용하면 이 딕셔너리를 생략하고 고정된 속성만을 가진 객체를 만들 수 있다.
```python
class Person:  
    __slots__ = ['name', 'age']  
  
    def __init__(self, name, age):  
        self.name = name  
        self.age = age
```

`장점`
- 메모리 사용 감소 : `__dict__`를 사용하지 않기 때문에 메모리 사용량이 줄어듦
- 성능 향상 : 메모리 사용이 최적화되어 객체 생성 및 속성 접근 속도가 빨라짐
`제한사항`
- 고정된 속성 : `__slots__`에 명시된 속성 외 다른 속성은 추가할 수 없음
- 상속 제한 : `__slots__`를 사용한 클래스를 상속할 때 자식 클래스도 `__slots__`를 명시적으로 정의해야 함
- 호환성 문제 : 일부 라이브러리나, 기능과의 호환성에 문제가 있을 수 있음
`상속`
- `__slots__`를 사용한 클래스를 상속할 때는, 자식 클래스에서 `__slots__`를 추가로 정의할 수 있음. 이 경우에, 부모 클래스의 `__slots__`를 자식 클래스에 포함시켜야 함
```python
class Person:  
    __slots__ = ['name', 'age']  
  
    def __init__(self, name, age):  
        self.name = name  
        self.age = age  
  
class Doctor(Person):  
    __slots__ = ['pay']  
      
    def __init__(self, name, age, pay):  
        super().__init__(name, age)  
        self.pay = pay
```

**추상클래스**
- 추상클래스는 인스턴스화할 수 없다. 서브 클래스는 추상클래스의 메서드 구현이 강제된다.
- abc 모듈을 사용해서 정의한다.
```python
from abc import ABC, abstractmethod  
  
class Animal(ABC):  
    @abstractmethod  
    def sound(self):  
        pass  
  
  
class Dog(Animal):  
    def sound(self):  
        return "bark"
```
- 추상메서드에는 `@abstractmethod` 데코레이터를 사용해야 한다.

**프로토콜**
- 정식으로 클래스의 메서드나 속성을 정의하지 않고도 특정 동작을 따르는 객체를 만들기 위한 개념으로, 타입 힌팅이나 정적 타입 검사를 더 효과적으로 할 수 있다.
- `typing` 모듈의 `Protocol` 클래스를 사용한다.
```python
class Swimmer(Protocol):
    def swim(self) -> None:
        ...

class Fish:
    def swim(self) -> None:
        print("Fish is swimming")

class Human:
    def swim(self) -> None:
        print("Human is swimming")

# 사용 예시
def let_it_swim(swimmer: Swimmer) -> None:
    swimmer.swim()

fish = Fish()
human = Human()

let_it_swim(fish)  # Fish is swimming
let_it_swim(human)  # Human is swimming
```
`duck typing` ?
- "오리처럼 걷고 오리처럼 소리내면 그것은 오리일 것이다" 라는 철학에 기초한다. 객체가 특정 인터페이스를 구현했는지 여부는 확인하지 않고, 필요한 메서드나 속성을 갖고 있으면 사용한다.
- `let_it_swim`의 argument로 `Swimmer` 타입의 객체가 들어와야 하지만, `human`과 `fish`가 들어온다. 하지만, `TypeError`는 발생하지 않았다. `human`, `fish` 둘 다 `swim()` 메서드를 갖고 있었기 때문이다.


**context manager**
- 코드 블록을 진입하고 나올 때 자동으로 설정 및 정리 작업을 수행하는데 사용된다.
- 일반적으로 `with`문과 함께 사용되며, 자원 관리(파일/ 네트워크 연결/ 락 등)에 유용하다
- `__enter__` 및 `__exit__` 메서드를 사용하거나 `contextlib` 모듈의 데코레이터를 사용하는 방법이 있다.
```python
with open('example.txt', 'w') as file:
	file.write('hello world')
# with 문이 끝나면 파일이 자동으로 닫힌다.
```


`__enter__` & `__exit__` 를 사용한 구현
```python
class ContextManager:  
    def __enter__(self):  
        print("this is enter")  
        return 3  
    def do_something(self):  
        print("using resource")  
    def __exit__(self, exc_type, exc_val, exc_tb):  
        print("this is exit")  
  
  
with ContextManager() as ctx:  
    print(ctx + 5)

---
this is enter
8
this is exit
```
- `__enter__` : `with` 문에 진입할 때 호출되는데, 임의의 값을 return시킬 수 있다.
- `__exit__` : `with` 블록을 벗어날 때 호출된다. 네 개의 인자는 각각 예외의 타입/값/트레이스백을 나타낸다. 예외가 발생하지 않으면 `None`이 전달된다.

`contextlib` 모듈을 사용한 구현
```python
from contextlib import contextmanager  
  
@contextmanager  
def context_manager():  
    print("this is enter")  
    yield # 함수의 호출을 외부로 양보한다는 의미
    print("this is exit")  
  
with context_manager():  
    print("using resource")
```