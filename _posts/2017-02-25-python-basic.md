---
title: 파이썬 기초 간단 정리 
author: 강성우
layout: post
category: [Python]
tag: [python]
---

이 문서에서는 지금 까지 파이썬을 공부 하면서 필요하거나 중요한 내용들 위주로 정리 하였다. 어느 언어라던지 반복적인 학습을 통해서 문법을 익히고 그 다음에 간단한 프로젝트를 하면서 문법에 익숙해지는 것이 좋다고 생각 한다. 하지만 파이썬과 같은 동적 언어가 아닌 자바같은 정적 언어에 익숙해져있는 상태 라면 아무래도 파이썬을 익히기에 어렵게 느껴질 수는 있다. 대부분 이 문제는 문법에 국한되어 있는 경우가 많으며 이는 반복적인 학습을 통해서 익히는 방법 밖에 없다고 생각 한다. 

하지만 나는 머리가 그렇게 좋지 않아서 기존에 작성한 코드나 문법을 일일히 찾아가면서 체크 하고 개발 하는일이 많은 편 이다. 특히 익숙해지지 못한 언어 에서는 더욱 더 그렇다. 기존에 작성한 스터디 정리 내용을 일일히 뒤져보다가는 너무 시간을 잡아 먹어서 각 챕터별로 매우 간소하게 문법과 그 예제 소스로 간단히 정리 하였다. 

### 1. 원시 자료형, 문자열 

###### 1.1 출력 메소드 

```py 
print("Hello World")
```

###### 1.2 원시 자료형의 선언 

```py 
# integer
intvalue = 10

# long / big number
# 원래 큰 숫자 뒤에 `L`키워드를 사용 해야 했지만 이젠 사용 하지 않아도 문제 없는거 같다. 
longValue = 98765432109876543210987654321

# float
floatValue = 0.12345678901

# Octal number
octalValue = 0o1234

# Hexa number
hexaValue = 0x1234
```

###### 1.3 연산 

```py 
# 사칙 연산
a = 10
b = 25
c = 0.55

print(a + b)
print(b - a)
print(a * c)
print(b / a)

# 제곱 연산 (x 의 y 제곱) `**`
print(a ** b)
print(b ** a)

# 나머지 연산 `%`
print(b % a)

# 나누고 나머지를 버림 `//`
# 음수에 이 연산자를 사용 할 경우 조심 해야 한다.
print(b / a)
print(b // a)
```

###### 1.4 자료형의 판단

-  자료형이 `True` 혹은 `False`인지 판단 하는 방법 

자료형 | 예시 값 | 결과 
--- | --- | --- 
문자열 | `"string"` | True 
문자열 | `""` | False 
리스트 | `[1, 2, 3, 4]` | True 
리스트 | `[]` | False 
튜플 | `()` | False 
딕셔너리 | `{}` | False 
숫자 | `n != 0` | True 
숫자 | `n == 0` | False 
숫자 | `None` | False 

- 자료형이 서로 같은지 여부를 판단 하는 방법 (인스턴스의 체크)

```py 
a = 10
b = 10
c = "Hell Korea"
d = "KangSungWoo"
e = b

print(a is b)	# true
print(a is c)	# false
print(b is a)	# true
print(c is d)	# false
print(e is b)	# true
```

###### 1.5 문자열 처리 

```py
montyPython = "Monty python's Flying Circus"
```

- 문자열 일정 구역 자르기 

```py
print(montyPython[0:14])
print(montyPython[15:21])
print(montyPython[:])       # 0 ~ 
```

- 문자열 포매팅 

```py
print("Hello, %s World!!" % "Python")   # 문자열 대입
print("Hello, %d World!!" % 1000)       # 숫자 대입
print("Hello, %d World!!" % b)          # 숫자 변수 대입

# 여러개 대입
print("Hello, %s World!! %d" % ("Python", a))
```

- 대입 연산자의 종류 들 

포매팅에 사용되는 대입 연산자는 다음과 같다. 필요에 맞추어 쓰면 될 것 이다. 

포맷 | 설명 
--- | --- 
%s | 문자열 
%c | 문자 
%d | 정수 
%f | 부동소수
%o | 8진수 
%x | 16진수 


### 2. 컬렉션 

###### 2.1 List 

- 리스트 기본 

```py
evenNumbers = [2, 6, 8, 12, 16, 22]

list1 = []
list2 = [1, 2, 3]
list3 = ['kang', 'kim', 'park']
list4 = ['kim', 100, 0.15, 'lee']
list5 = [1, ['kim', 'jung', 'lee'], 2, 3]

list2[1] = [100]
list2[1:2] = [100, 200]
list2[0] = ['abc', 'def'] 

# get lenth of List 
length = len(list2)

# delete one element at index
del list2[0]

# delete multiple elements of index
list3[0:2] = []

# delete element of value 
list4.remove('lee')

elist = [1, 2, 3, 4, 5, 6]
element = elist.pop()
```

- 리스트의 연산 

```py
aList = [1, 2, 3, 4]
bList = [5, 6, 7, 8]

# 리스트 더하기
print(aList + bList)

# 리스트 반복
print(aList * 2)
print(bList * 3)
``` 

- 리스트의 정렬 

```py 
numList = [4, 2, 12, 8, 5, 1, 3]
numList.sort()
```

- 리스트 리버스 

```py 
elist = [1, 2, 3, 4]
elist.reverse()
```

- 리스트에서 원소 찾고 찾았을 경우 해당 하는 인덱스 얻기 

```py 
elist = [11, 22, 33, 44]
foundIndex = elist.index(22))

slist = ['a', 'b', 'c', 'd']
foundIndex = slist.index('b')
```

- 리스트 중간에 원소 삽입 

```py 
elist = [1, 2, 3, 4, 5]
elist.insert(0, 999)        # (index of insert, insert value)
```

- 특정 원소의 갯수 얻기 

```py 
testlist = [1, 2, 1, 3, 1, 4, 3, 5]
print(testlist.count(1))
```

###### 2.2 Tuple 

- 기본 사용 법 

```py
t = (1, 2, 3, 4, 5)
print(t) 
```

- 리스트와 튜플의 다른 점 

리스트는 생성 하고 난 뒤 원소들을 추가, 삭제, 수정 할 수 있음. 하지만 튜플을 불가능. 어떠한 목록의 크기를 이미 모두 알 고 있고 그 크기가 영원히 변하지 않는 다면 튜플을 사용 하고 그렇지 않다면 리스트를 사용 하면 된다. 

###### 2.3 Dictionarie 

- 기본 구조 

**Key**와 **Value** 쌍으로 이루어진 구조. 아래 예제 참고 

```py
dic = {'name':'Kang', 'email':'kang1010@google.com', 'phone':'01012345678'}
```

딕셔너리 `dic`를 테이블로 표현하면 아래와 같다. 

key | value
--- | --- 
name | Kang 
email | kang101@google.com 
phone | 01012345678 

- 딕셔너리의 key-value 사용 법 

```py 
dic = {'name':'Kang', 'email':'kang1010@google.com', 'phone':'01012345678'}

# add `key=value` 
dic['gender'] = 'male'

# delete `key` and value 
del dic['name']

# search key and get value 
dic['email']  

# make key or value to List
dicKeys = list(dic.keys())
dicValues = list(dic.values())

# make key=value to Tuple 
dicItems= dic.items()
# ex) dict_items([('name', 'Kang'), ('phone', '01012345678'), ('email', 'kang1010@google.com')])

# get value of key 
dic.get('email')

# if has key in Dictionarie
'email' in dic
'address' in dic
```

###### 2.4 Set 

파이썬 2.4 에서부터 지원하는 중복을 허용하지 않으며 입력 순서가 없는 자료들의 집합. 

- 기본 구조 

```py
s = set([1, 2, 3, 4])       # result {1, 2, 3, 4}
s = set("HelloWorld")       # result {'d', 'e', 'o', 'l', 'W', 'H', 'r'}

# add element to Set 
s.add(5)

# add multiple element to Set 
s.update([6, 7, 8])

# delete value 
s.remove(6)
```

### 3. 제어 처리 

###### 3.1 if-else

```py 
value = 90
if value == 100:
    print("값이")
    print("100 이다")
elif value == 90:
    print("값이 90 이다")
elif value == 99:
    pass
else:
    print("값이 100이 ")
    print("아니다")

print("프로그램 종료")
```

if, elif, else 문과 기존 소스의 구분은 **들여쓰기**로 구분 한다. 

- 자료형의 판단 

자료형 | `True` | `False` 
--- | --- | --- 
숫자 | 0 이 아닌 모든 숫자 | `0`
String | 길이가 0보다 큰 문자열 | `""` (길이가 0인 문자열)
List | 원소가 하나 이상 존재하는 리스트 | `[]` (원소가 없는 리스트)
Tuple | 원소가 하나 이상 존재하는 튜플 | `()` (원소가 없는 튜플) 
Dictionarie | 키-원소 쌍이 하나 이상 존재 하는 딕셔너리 | `{}` (키-원소 쌍이 없는 딕셔너리) 

- 비교 연산자 

비교연산자 | 설명 
--- | --- 
`x < y` | `x` 값이 `y`보다 작다. 
`x > y` | `x` 값이 `y`보다 크다. 
`x == y` | `x` 값과 `y`이 같다. 
`x != y` | `x` 값과 `y`이 같지 않다. 
`x <= y` | `x` 값이 `y`와 같거나 작다. 
`x >= y` | `x` 값이 `y`와 같거나 크다. 

- and, or, not 연산자 

연산자 | 설명 
--- | --- 
`x and y` | x 와 y 둘 다 `True` 일 경우에만 `True`가 된다. 만약 둘중 하나라도 `False`가 있다면 `False`가 된다. 
`x or y` | x 와 y 둘 중 하나 이상이 `True` 일 때만 `True`가 된다. 하지만 둘 다 `False`라면 `False`가 된다. 
`not x` | x 가 `False`라면 `True` 이다. 

```py 
value = 100
strValue = ""
if value == 100 and value != 900:
	print("값은 정확히 100 이면서 900은 절대로 아니다.")
if value != 50 or value >= 100:
	print("값은 50이 아니거나 100보단 크거나 같다.")
if not value:
	print("값인 False(0)이 아니다.")
if not strValue:
	print("문자열이 현재 비어 있다.")
if not value == 900:
	print("값이 900 이 아니다.")
```

- in, not in 연산자 

`in`과 `not in` 비교 연산자는 어떠한 리스트나 튜플, 딕셔너리 와 같은 자료 구조에 원소로 존재 하는지 혹은 존재하지 않는지 확인 하는 연산자 이다. 사용 예는 아래 예제 소스를 확인 해 보자. 

```py 
print("1 은 [1, 2, 3] 리스트 안에 존재 하는가? %s " % (1 in [1, 2, 3]))
print("`A` 는 (`K`, `C`, `D`) 튜플 안에 존재 하는가? %s " % ('A' in ('K', 'C', 'D')))
print("500 은 [100, 200, 300] 리스트 안에 존재하지 않는가?  %s " % (500 not in [100, 200, 300]))
print("`Kang` 은 [`Kim`, `Kang`, `Jang`] 리스트 안에 존재하지 않는가? %s " % ("Kang" not in ["Kim", "Kang", "Jang"]))
print("'D'는 'PythonDjango 문자열에 존재 하는 문자인가? %s " % ('D' in "PythonDjango"))
```

###### 3.2 while 

- 기본 구조 

```py
index = 1
while index <= 10:
	if index < 10:
		print("index 값은 %d 입니다." % index)
	else:
		print("index 값이 %d 입니다. 마지막 숫자 입니다." % index)
	index += 1
```

- continue, break 

```py 
index = 0
while index <= 10:
	if index == 2:
		index += 1
		continue 	# index 가 2일땐 index를 증가 시키고 다음 반복으로 진행
	elif index == 9:
		print("index 가 9일땐 강제 종료 합니다. ")
		break
	else:
		print("index 값은 %d 입니다." % index)
	index += 1
```

###### 3.3 for 

- 기본 구조 

```py 
scoreList = [32, 11, 94, 26, 86, 54, 62, 80, 73]
for score in scoreList:
	print("점수는 %d 입니다." % score)

tupleList = [(5, 10), (80, 42), (22, 19)]
for (x, y) in tupleList:
	print("%d + %d = %d" % (x, y, x+y))

# (0, 1, 2, 3, 4) 
intList = range(5)
for value in intList:
	print("%d" % value)

sum = 0;
for value in range(1, 101):
	sum += value
print("1 부터 100까지의 합은 %d 입니다." % sum)

# 구구단 
for x in range(2, 10):
	for y in range(1, 10):
		print("%d * %d = %d" % (x, y, x*y))
```

###### 3.4 List comprehension (리스트 내포)

리스트 원소에 for 문을 이용 하여 데이터를 세팅 하는 방법. 

```py 
# 일반적인 방법 
value = []
for i in range(5):
	value.append(i * 2)
print(value)

# list comprehension 기법 을 사용 한 예제 
value = [i * 2 for i in range(5)]
print(value)
```

기본 사용법은 `리스트 = [대입변수의표현 for 대입변수 in (for문)(if문)]`이다. 

```py 
# list comprehension 두번째 예제 (0 부터 14까지 숫자들 중 짝수만)
value = [i for i in range(15) if i % 2 == 0]
print(value)

# 리스트 내 튜플의 list comprehension 세번째 예제 (n, n*n), ...
value = [(x, x*x) for x in range(5)]
print(value)
```

### 4. Method 

- 메소드의 기본 구조 

```py 
def 함수이름(인수1, 인수2, 인수3 ...):
    # 실행 내용 
    return 결과값
``` 

사용 예 

```py 
def sum(x, y):
	return x + y

print(sum(12, 34))
print(sum(100, 300))
```

- 여러개의 인수를 가진 메소드의 구현 

```py 
# def sumOfAllNumbers(a, b, c, d, e, f, g): ...

def sumOfAllNumbers(*nums):
	sum = 0
	for n in nums: sum += n
	return sum
print(sumOfAllNumbers(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13))

def sumOfAllNumbers(value, *nums):
	sum = value
	for n in nums: sum += n
	return sum
print(sumOfAllNumbers(10000, 10, 20, 30, 40, 50))
```

- 메소드 매개변수의 초기화 

입력받는 매개변수가 존재하지 않을때 기본값으로 초기화 해줄 값을 명시 할 수 있다. 그렇게 사용 할 경우 사용 하는 모든 매개변수의 값에 초기화를 해주어야 한다. 만약 하나라도 해주지 않았을 경우 예외가 발생 한다. 

```py 
def divideValues(x = 1, y = 1):
	return x / y

print(divideValues(200, 100))
print(divideValues(30))
print(divideValues())
```

### 5. Class  

- 기본 형태 

```py 
class Calculator:
	x = 0
	y = 0

	# 클래스의 생성자
	def __init__(self, x, y):
		self.x = x
		self.y = y

	# x 와 y 의 합을 얻는 메소드
	def add(self):
		return self.x + self.y

	# x 에서 y 를 뺀 값을 얻는 메소드
	def minus(self):
		return self.x + self.y

	# x 와 y를 나눈 값을 얻는 메소드 (x 나 y가 둘 중 하나라도 0 일경우 0을 반환)
	def divide(self):
		if self.x == 0 or self.y == 0:
			return 0
		return self.x / self.y

	# x 와 y 를 곱한 값을 얻는 메소드
	def multiply(self):
		return self.x * self.y

	# 인스턴스 상태를 출력할 메소드 
	def __str__(self):
		return "x = " + self.x + ", y = " + self.y


calc = Calculator(21, 3)
print("%d + %d = %d" % (calc.x, calc.y, calc.add()))
print("%d - %d = %d" % (calc.x, calc.y, calc.minus()))
print("%d / %d = %d" % (calc.x, calc.y, calc.divide()))
print("%d * %d = %d" % (calc.x, calc.y, calc.multiply()))
```

- 클래스의 상속 

```py 
class ScientificCalculator(Calculator):

	def mod(self):
		return self.x % self.y

	def squarex(self):
		return self.x ** 2

	def square(self):
		return self.x ** self.y

sincalc = ScientificCalculator(14, 3)
print("%d + %d = %d" % (sincalc.x, sincalc.y, sincalc.add()))
print("%d mod %d = %d" % (sincalc.x, sincalc.y, sincalc.mod()))
print("%d square %d = %d" % (sincalc.x, sincalc.y, sincalc.square()))
print("%d square 2 = %d" % (sincalc.x, sincalc.squarex()))
```

`Calculator` 클래스를 상속한 클래스의 예제. 

```py 
class ScientificCalculator(Calculator):
	z = 0

	def __init__(self, x, y, z):
		Calculator.__init__(self, x, y)
		self.z = z

sincalc = ScientificCalculator(14, 3, 12)		
```

생성자를 구현 하고 부모 클래스의 생성자를 호출 하여 값을 초기화 하는 생성자의 예. 

- 메소드 오버라이딩 

일반적인 방법과 동일 하다. 부모 클래스를 상속한 클래스에서 부모 클래스의 메소드를 재정의 하여 구현 하고 사용 하면 된다. 

- 연산자 오버로딩 

사칙연산에 사용 되는 `+, -, *, /`을 이용 하여 클래스 인스턴스간의 연산에 해당 하는 기능을 재정의 하는 기법 이다. 

메소드 | 연산자 | 사용 예 
--- | --- | --- 
`__add__(self, other)` | + 이항 | `A + B, A += B` 
`__pos__(self)` | + 단항 | `+A` 
`__sub__(self, other)` | - 이항 | `A - B, A -= B` 
`__neg__(self)` | - 단항 | `-A` 
`__mul__(self, other)` | * | `A * B, A *= B` 
`__truediv__(self, other)` | / | `A / B, A /= B` 
`__floordiv__(self, other)` | // | `A // B, A //= B` 
`__mod__(self, other)` | % | `A % B, A %= B` 
`__pow__(self, other)` | `pow()`, ** | `pow(A, B), A ** B` 
`__lshift__(self, other)` | << | `A << B, A <<= B` 
`__rshift__(self, other)` | >> | `A >> B, A >>= B` 
`__and__(self, other)` | & | `A & B, A &= B` 
`__xor__(self, other)` | ^ | `A ^ B, A ^= B` 
`__or__(self, other)` | `|` | `A | B, A |= B` 
`__invert__(self)`, | `~` | `~A` 
`__abs__(self)` | `abs()` | `abs(A)`

- Enum 클래스 

파이썬 3.4 버전 이상에서 사용 가능한 기법 으로서 `Enum`클래스를 상속하여 구현 한다. 

```py 
from enum import Enum

class ViewState(Enum):
	NORMAL = 1
	DISABLED = 2
	LOADING = 3

viewState = ViewState.NORMAL
```

- 추상 클래스 

```py 
from enum import Enum
from abc import ABCMeta, abstractmethod

# Shape type enum
class ShapeType(Enum):
	Rectangle = 1
	Oval = 2

# Shape 추상 부모 클래스
class Shape(metaclass=ABCMeta):
	type = ShapeType.Rectangle
	x = 0
	y = 0
	width = 0
	height = 0

	def __init__(self, type, x, y):
		self.type = type
		self.x = x
		self.y = y

	# 추상 메소드
	@abstractmethod
	def drawShape(self):
		pass

	def setSize(self, width, height):
		self.width = width
		self.height = height

# Rectangle 클래스
class Rectangle(Shape):
	def __init__(self, x, y):
		super().__init__(ShapeType.Rectangle, x, y)

	def drawShape(self):
		print("%s : x = %d, y = %d, width = %d, height = %d" % (self.type, self.x, self.y, self.width, self.height))

# Circle 클래스
class Circle(Shape):
	radius = 0

	def __init__(self, x, y, radius):
		super().__init__(ShapeType.Oval, x, y)
		self.radius = radius

	def drawShape(self):
		print("%s : x = %d, y = %d, width = %d, height = %d, radius = %d" % (self.type, self.x, self.y, self.width, self.height, self.radius))

rect = Rectangle(50, 30)
rect.setSize(100, 100)
rect.drawShape()

circle = Circle(20, 20, 5)
circle.setSize(60, 60)
circle.drawShape()
```

추상 클래스에서는 `metaclass=ABCMeta`라는 메타 데이터를 설정 하여 추상 클래스임일 컴파일러에 알린다. `metaclass`는 `__metaclass__`메소드의 재정의로서 싱글턴 패턴의 선언과 같은 형태 이다. 

### 6. Module 

다른 py 파일을 가져와서 모듈로 사용 하는 방법. 이 때 `import` 구문을 이용 해서 선언 해서 사용 한다. 

- 기본적인 모듈의 사용 방법 

```py 
# module calc.py
def sum(x, y):
	return x + y

def minus(x, y):
	return x - y

def divide(x, y):
	if x == 0 or y == 0: return 0
	return x / y

def multiply(x, y):
	return x * y


# main.py
import calc

print(sum(10, 20))
print(minus(100, 50))
```

- 다른 디렉터리나 네임 스페이스의 활용 기법 

`from 모듈이름 import 대상클래스_혹은_메소드명` 의 형태를 갖는다. ㄷ

```py 
# main.py 
import calc
from calc import sum
from calc import divide

print(sum(10, 30))
print(divide(12, 6))
print(calc.multiply(4, 8))
```


### 7. Package 

계층 구조에서 하위 디렉토리로 설정된 패키지에는 `__init__.py`라는 파일을 보유 하고 있을 경우 패키지 디렉터리임을 파이썬 컴파일러가 알 수 있게 된다. 이를 통해서 패키지의 모듈로 가져 오기 위해서 `from ... import ... `구문을 이용 하여 네임스페이스로 접근하여 사용 한다. 

### 8. Exception 

- 기본 예외 처리 

```py 
num = 23 

try: 
	num / 0

except ZeroDivisionError:
	print("Error 'ZeroDivisionError'")
```

`try` 블럭에서 예측 가능한 혹은 시나리오상 발생하면 안되는 예외를 체크 하고 예외 발생시 `except:`구문에서 처리 한다. 

- 예외발생시 예외의 정보를 변수로 받기 

```py 
num = 23 

try: 
	num / 0

except ZeroDivisionError as error:
	print(error)
```

- 예외에 대한 추가적인 처리 

```py
num = 23

try:
	num / 5

except ZeroDivisionError as error:
	print(error)

else:
	print("정상적인 케이스 입니다.")	# 예외가 발생하지 않았을 경우 

finally:
	print("모든 실행 구문을 완료 했습니다.")	# 에외가 발생하거나 발생하지 않아도 항상 실행되는 구문 
```

- 여러개의 예외를 검사 하는 방법 

```py 
try: 
	# 실행 내용 

except: 오류1 as error1:
	# 오류 1에 대한 처리 구문 1

except: 오류2 as error2:
	# 오류 2에 대한 처리 구문 2
```

혹은 아래와 같이 할 수도 있다. 

```py 
try:
	# 실행 내용 

except: (오류1, 오류2) as error:
	# 오류 1 혹은 2에 대한 공통 처리 구문 
```

- 발생한 예외를 무시하는 방법 

```py 
try:
	# 실행 내용 

except: 오류 as error:
	pass
```