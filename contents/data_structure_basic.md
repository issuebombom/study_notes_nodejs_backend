# 자료구조

자료구조 공부를 시작하면 배열, 리스트, 링크드리스트, 해시, 체이닝, 오픈 어드레스, 큐, 스택, 트리 등 여러 용어를 배우게 된다.  
하지만 이것들이 무엇인지 하나씩 아는 것도 중요하지만 이해를 돕기 위해 확실히 집고 넘어가야 할 부분이 있다.

## 추상 자료형(abstract data type, ADT) vs 자료 구조(data structure)

**추상자료형**은 쉽게 말해 파이썬의 list, dictionary 그리고 자바스크립트의 array, object 등을 말한다.  
이는 자료를 저장하기 위해 사용되는 일종의 `도구`, `기능`에 해당한다.  
우리가 자바스크립트의 array에 대해서 접근할 때 string이나 number 등을 추가하거나 수정, 삭제하기 위한 도구로서 바라볼 것이다.  
그래서 우리는 array에 숫자 3을 추가하고 싶다면 .push(3) 메서드를 사용할 것이다.  
하지만 구체적으로 array.push(3) 메서드를 적용했을 때 `내부적으로 어떻게 작동하는지에는 관심이 없었을 것이다.`

**자료구조**가 바로 `내부적으로 어떻게 작동하는지에 대한 이야기에 해당한다.`  
파이썬의 리스트(추상 자료형) 같은 경우 동적배열(자료구조) 방식으로 `구현`되어 있다.  
그러므로 자료구조는 어떻게 '구현'하였는가에 대한 이야기라고 볼 수 있다.

정리하자면 아래 문구와 같이 추상자료형과 자료구조를 축약할 수 있다.  
`파이썬에서 리스트(추상자료형)는 내부적으로 동적배열(자료구조) 방식으로 작동하고, 딕셔너리(추상자료형)는 해시테이블(자료구조) 방식으로 작동한다.`

> `참고`  
> list - 동적 배열  
> Queue - 더블리 링크드 리스트  
> Stack - 동적배열, 더블리 링크드 리스트  
> dictionary, map, set - 해시 테이블

이 두 가지 관점이 나뉘어 있다는 점을 기억하고서 아래 내용이 자료구조에 대한 이야기인지 아니면 특정 자료구조가 적용된 추상 자료형에 대한 이야기인지 순간순간 판단하면서 읽어나가길 바란다.

## 데이터 저장 방식

CS 공부를 조금이라도 했다면 데이터가 통상적으로 RAM이라 불리는 메모리에 저장된다는 것을 알고 있을 것이다.  
메모리는 길게 `한 줄로 늘어진 '방'`과 같다고 생각하면 편하다. 각 방을 구분하기 위해 '방 번호'가 배정되어 있어 어떤 데이터가 몇 호실 방에 보관되어 있는지 기록해두어 필요할 때 찾아쓰는 구조이다.

위에서 잠깐 언급했듯 메모리는 한 줄로 늘어진 방과 같다고 했는데, 이 말을 쓰는 이유는 방을 여러 개 대여하는 상황에서는 통상적으로 여러 개의 방이 일렬로 따닥따닥 붙은 방들을 빌려주는 방식을 취하기 때문이다.  
만약 어떤 데이터를 저장하는데 필요한 방의 개수가 4개라면 101, 102, 103, 104호 방을 빌려주는 구조를 갖고 있으며, 101, 213, 104, 315와 같이 뿔뿔이 흩어진 방을 빌려주지 않기 떄문이다.

그 이유는 효율성에 문제가 되기 때문이다. 데이터 저장을 위해 방을 '빌린 사람'의 입장에서 생각해보자, 한 사람이 데이터를 저장하기 위해 방을 빌리려는데 방 한개로는 어림도 없는 크기라고 하자. 그렇다면 어떻게든 그 데이터를 나누어서 여러 방에 저장하는 방식을 취해야 할텐데, 근본적으로 그 데이터는 한 그룹에 속한다. 그러므로 나눠진 데이터는 서로 연관성이 있다고 볼 수 있고, 데이터를 사용할 때 같이 찾게 될 확률이 높을 것이다. 그러므로 저장된 방이 일렬로 나열되어 있어여 바로 옆방, 그리고 그 옆방에 데이터가 있어야 찾기가 효율적일 것이다. 쉽게 말해 시간이 얼마 걸리지 않는다는 말이다.

> `정리`
> 데이터는 메모리(방)에 저장되고, 각 메모리에는 주소(방 번호)가 있어 데이터를 보관하고 찾는데 활용된다.  
> 여러 데이터를 저장할 경우 각 메모리 공간(방)이 일렬로 나열된 형태를 빌려 저장한다.

## 레퍼런스

프로그래밍을 하면서 변수, 리스트에 데이터를 저장한다는 개념이 체득되어 있을 것이다. 하지만 엄연히 말해 `변수, 리스트는 우리가 자료에 접근할 수 있도록 돕는 도구(레퍼런스)에 해당`한다는 점을 기억하자.  
실제로 파이썬에서 name = 'apple' 로 변수 지정 후 id(name)을 print 해보면 apple이라는 자료가 저장된 메모리 주소(ex. 140349163284432)를 획득할 수 있다. 하지만 우리가 이 개연성 없는 메모리 주소를 기억할 수 없기 때문에 name이라는 변수를 통해 해당 메모리에 접근하도록 변수를 레퍼런스로 활용하는 것이다. 그러므로 변수가 자료 그 자체에 해당하지 않는다는 점을 기억하자.

## 배열(array)과 리스트(list)의 차이

배열은 자바스크립트의 배열이 아닌 `C 언어의 배열`을 말하며 리스트는 `파이썬의 리스트`를 말한다. 이 둘은 추상 자료형을 말하는 것임을 기억하길 바란다.

### 배열(array)

C 언어에서는 배열에 데이터를 저장하고자 할 때 `데이터 크기를 고려한 배열을 생성해야 한다.` 만약 [1, 2, 3, 4] 이렇게 총 4개의 정수를 담을 계획이라면 4byte의 공간을 쓰는 정수 4개를 담기 위해 총 16byte 크기의 배열 생성을 고려해야 한다는 것이다. 왜냐하면 한번 생성된 배열은 크기를 유동적으로 확장할 수 없기 때문이다. 즉 정수를 4개 이상 담을 수 없게 된다.

또한 `배열 생성 시 동일한 데이터 타입만 담을 수 있다.` 정수 배열에는 오직 정수만 담을 수 있으며, 문자열이나 소수를 섞어서 담을 수 없다는 얘기다. 그래서 애시당초 배열 생성 시 이 점이 확정된다.

이처럼 배열 생성 단계부터 신경써야 하는 C 언어 배열의 강점은 바로 `빠른 데이터 접근`과 `메모리 낭비를 최소화`할 수 있다는 점이다.

먼저 빠른 접근이 가능한 이유에 대해 살펴보자. 위에서 언급했듯이 일렬로 나열된 메모리를 빌린다면 그 주소도 일렬로 나열된다는 말이 된다.  
우리가 빌린 16byte의 주소가 1000에서 1015라고 가정하자. 그렇다면 정수 1은 1000 - 1003을 차지하고, 정수 2는 1004 - 1007을 차지할 것이다. 4byte의 공간이 필요한 데이터를 4개 담았으니 각 정수가 저장된 주소의 간격도 4가 된다.  
이 규칙으로 인해 우리는 첫 메모리의 주소만 기억한다면 다른 데이터의 주소에도 `$1000 + 4 * i$` 공식으로 빠르게 접근이 가능하게 된다.  
[1, 2, 3, 4] 배열에서 인덱스 3에 해당하는 정수 4에 접근하려고 한다면 해당 공식의 i에 3을 대입하면 정수 4가 시작되는 메모리 주소를 쉽게 구할 수 있다. 이 식의 연산속도를 시간복잡도 $O(1)$에 해당하는 속도로 매우매우 빠른 속도에 해당한다. 어떤 주소에 `접근`하고자 하더라도 $1000 + 4 * i$ 공식만 있으면 일정한 빠른 속도를 보장하는 것이다.

이게 가능한 이유는 `데이터를 선형적으로 저장`했다는 점, 사전에 `같은 타입의 데이터만 담도록 구성`했기 때문이다.

> 이러한 배열 방식을 `정적 배열`이라 한다.  
> 즉, C 언어의 배열은 정적 배열 방식이 적용되어 있다.

### 리스트(list)

파이썬의 리스트, 자바스크립트의 배열이 해당 추상 자료형에 해당한다. 대부분 파이썬과 자바스크립트 언어에 익숙할 것이므로 해당 자료형의 특징을 잘 알고 있을 것이라 생각한다.

> 1. 리스트를 생성할 때 크기를 고려하지 않아도 된다.
> 2. 리스트에 담을 수 있는 데이터 타입에 제한이 없다.

이게 가능한 이유는 `1. 리스트는 동적 배열이 적용`되어있고, `2. 자료를 랜덤한 메모리 주소에 배정`하기 때문이다.

동적배열은 내부적으로 알아서 배열의 크기를 정해주고, 정해진 배열의 크기가 가득차면 더 큰 배열로 확장해주는 방식을 말한다. 구체적으로는 어떻게 작동하는지 아래에서 살펴보자

> `동적배열 작동 방식`
>
> 1. array = [1, 2, 3, 4] 명령을 통해 배열을 생성한다.
> 2. 내부에서는 일반적으로 총 8개(4\*2)의 정수를 담을 수 있는 메모리를 확보한다.
> 3. append를 통해 데이터를 추가해 나가다가 8번째 데이터를 append하게 되었다.
> 4. 내부에서는 총 16개(8\*2)의 정수를 담을 메모리를 신규 확보한다.
> 5. 기존 배열에 담긴 정수 8개를 신규 확보한 메모리에 복사한다.
> 6. array변수가 신규 메모리 주소를 향하도록 한다.
> 7. Garbage Collector가 이전 메모리를 수거한다.

내부를 살펴보면 실제 메모리 공간을 확보하고 확장하는 방식이 정적 배열 방식의 틀에서 크게 벗어나지 않는 것을 알 수 있다. 단지 정적 배열 방식에 유동성을 더해줬을 뿐이다.

### 배열 vs 리스트

C 언어가 파이썬, 자바스크립트보다 빠르다고 하는 이유 중 하나가 여기에 있다. 편의성에 있어서는 `C언어`가 어렵게 느껴지고 불편한 면이 있지만 `같은 타입의 데이터를 선형적으로 보관한다.`라는 점이 `탐색에 있어 얼마나 효율적`인지 위에서 알아보았다. `파이썬`의 관점에서는 랜덤 주소에 데이터를 보관하므로 당연히 `탐색 연산이 비쌀 것`이다. 또한 `데이터가 찰 때 마다 새로운 메모리 생성, 복사, 붙여넣기를 해야하는 연산이 추가`되므로 C 언어보다 전반적으로 느릴 수 밖에 없다.

## 배열의 탐색, 수정, 삭제

탐색은 데이터를 찾는 행위를 말하며 배열의 탐색은 일반적으로 처음부터 하나씩 순차적으로 찾는 방식을 뜻한다. 쉽게 말해 for문을 돌리는 것이다. 시간복잡도로 보자면 $O(n)$에 해당하며 정적 배열, 동적 배열 동일하다.

수정은 배열에서 인덱싱으로 특정 데이터에 접근하여 그 값을 바꾸는 것을 말한다. 찾아서 수정하는 게 아니라 직접적으로 해당 메모리에 접근하여 수정하는 것을 말하므로 시간복잡도는 $O(1)$에 해당한다. 이 또한 정적 배열, 동적 배열 동일하다.

삭제는 내부적으로 조금 재밌게 동작한다. 단순하게 생각했을 때 삭제는 특정 메모리에 접근하여 해당 데이터를 삭제, 메모리 공간을 축소시킨다고 생각할 수 있겠지만 그렇지 않다.  
[1, 2, 3, 4]의 배열에서 인덱스 2에 접근하여 3이라는 값을 삭제한다고 하자. 그러면 내부적으로는 3이라는 값을 4로 변경해버리는 방식을 취한다. 왜그런걸까?  
첫번째로 메모리는 일정 크기를 삭제해서 그 크기를 줄인다는 식으로 접근하지 못한다. 그저 더 작은 공간을 새로 확보해서 기존 데이터를 복붙한다는 개념을 접근해야 할 것이다. 하지만 이는 성능 측면에서 매우 비효율적이다.

그렇기에 해당 인덱스의 다음 인덱스값으로 대체하는 방식(덮어쓰기)으로 구현한 뒤 마지막 인덱스에 접근하지 못하도록 한다.  
예를 들어 파이썬에서 array=[1, 2, 3, 4, 5]가 있는데 array[2]를 삭제하면 내부적으로는 (1, 2, 4, 5, 5) 형태가 되고 마지막 인덱스 자리의 메모리에 접근하지 못하도록 한다.  
이는 상황에 따라 시간복잡도가 달라지게 된다. 0번째 인덱스 값을 삭제하면 나머지 값을 전부 바꿔줘야 하므로 `O(n)`이 걸리지만 마지막 인덱스 값을 삭제하면 `O(1)`이 걸린다. 참고로 C 언어와 같은 정적 배열은 기본적으로 삭제한 자리가 비워질 뿐 요소를 하나씩 당겨준다거나 하는 작업 조차 하지 않는다. 그러므로 `많은 삭제가 발생할 경우 정적 배열은 메모리 낭비를 초래`하게 된다.

이에 반해 정적 배열에서 삭제가 유리한 점 중 하나를 배열의 크기를 조절한다는 점이다. 만약 정수 4개를 저장하기 위한 배열 생성을 요청하게 되면 내부적으로 곱하기 2인 8개의 자리를 마련한다. (X2는 일반적인 경우에 해당하며 이 값은 기본 설정에 따라 다를 수 있다.)  
하지만 반대로 정수 4개를 저장하다가 3개로 줄어든다면 내부적으로 8개의 메모리 공간을 4개 짜리 공간으로 변경해버린다. 그러므로 연산 속도가 느릴지언정 메모리 낭비를 방지할 수 있는 것이다. 그래서 `삭제 연산에 있어서는 동적 배열이 정적 배열보다 메모리 사용 효율이 높은 것`이다.

---

## 링크드리스트(LinkedList)

링크드 리스트는 데이터가 메모리 주소를 기반으로 연결되는 방식의 자료 구조에 해당하며 데이터 추가, 삭제에 유리하다.

파이썬 링크드리스트는 아래 코드를 통해 구현 방식을 이해해 보자.

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None
        self.tail = None

    def append(self, data):
        new_node = Node(data)
        if self.head == None:
            self.head = new_node
            self.tail = new_node
        else:
            self.tail.next = new_node
            self.tail = new_node

# 데이터 3, 5, 7을 담는다고 했을 때
array = LinkedList()
array.append(3)
array.append(5)
array.append(7)
```

먼저 Node를 살펴보자. Node에는 인스턴스 변수로 data에 자료(데이터), next에 메모리 주소를 담는다. 그래서 각 노드에 데이터를 하나씩 담으면서 노드 간 연결을 위해 next 변수를 이용한다고 생각하면 된다.  
이 Node는 LinkedList 클래스의 append 메소드를 통해 생성되며, 이렇게 생성된 여러 Node는 head와 tail 변수를 통해 연결된 노드의 시작점과 끝점을 파악할 수 있게 한다.

원리를 간단하게 정리하자면 아래와 같다.

> LinkedList 원리  
> 첫 데이터를 첫번째 노드 클래스의 data 변수에 담는다.  
> 두번째 데이터를 두번째 노드 클래스의 data 변수에 담는다.  
> 첫번째 노드의 next 변수에 두번째 노드 메모리 주소를 담는다.  
> 이렇게 되면 첫번째 노드가 head가 되고 두번째 노드가 tail이 된다.

여기서 또 데이터를 추가하면 어떻게 될까? 세번째 노드를 생성하여 데이터를 담고, 두번째 노드의 next에 세번째 노드 메모리 주소를 담는다. 그리고 이제 노드의 끝 지점은 세번째 노드가 되었으므로 세번째 노드를 tail로 임명한다.

이러한 자료구조의 장점은 `데이터 추가, 삭제가 리스트(동적배열)보다 빠르다`는 점이다. 동적 배열은 데이터 추가, 삭제로 인해 메모리 공간을 변경하거나, 데이터를 한칸씩 뒤로 밀어야 할 때가 발생하는데 링크드리스트는 그런게 전혀 없기 때문이다.

하지만 접근에 있어서는 불리하다. 링크드리스트는 기본적으로 head와 tail 외에는 인덱싱 접근이 불가하기 때문에 무조건 head에서 부터 탐색해야 하기 때문이다. head 노드에서 시작하여 next를 토대로 다음 node로 이동하고, 내가 찾는 데이터가 맞는지 체크하고, 아니면 또 next...이러한 방식이다.

결론적으로 뒤에서 보게 될 head, stack처럼 어떠한 task를 순차적으로 처리해야하는 상황에서 사용하기에 유리하다. node의 가장 앞, 가장 뒤 데이터에 접근, 추가, 삭제 속도가 빠르기 때문이다.

## 해시 테이블

우리가 흔히 하는 key-value 접근을 말하며 key를 입력하면 해당하는 메모리 주소로 이동하여 value를 가져오는 방식이다. 이렇게 Key-value 쌍이 표에서 여러 행으로 쌓이게 되면 테이블 형태가 되다보니 이를 해시테이블이라 부른다.

그렇다면 문제는 어떤 데이터를 어느 주소에 저장했고, 어떻게 접근할 것인가이다. 여러 가지 접근 방법이 있지만 기본적으로는 `변수를 특정 함수 f(x)에 넣었을 때 메모리 주소가 출력되도록 한다.` 쉽게 말해 'apple'이라는 단어를 f(x)에 넣어서 1032라는 메모리 주소가 나오도록 할 수 있다면 변수와 f(x)를 토대로 해당 메모리에 데이터를 저장하거나 열람할 수 있다.
my_room_number = 234와 같은 방식이 내부적으로 이러한 방식으로 작동한다.

### 해시 함수

위와 같이 x값을 입력하면 메모리 주소를 내어주는 함수 f(x)를 해시 함수라고 한다. 이 함수는 다양한 알고리즘으로 구현할 수 있는데 가장 간단한 나누기 방식을 살펴보자

#### 나누기 방식

만약 213호에 사는 '김민수' 씨와 같은 정보를 데이터에 저장하기 위해 우리는 key=213, value='김민수'를 떠올릴 수 있을 것이다. 먼저 우리가 사전에 확보한 메모리주소의 범위가 0 - 199라고 하자. 우리는 주소 0에서 199까지만 쓸 수 있는 상황이다. 그렇다면 213을 총 주소 개수인 200으로 나누고, 나머지 값인 13을 메모리 주소로 적용할 수 있을 것이다.

그 밖에 여러 해시 알고리즘이 있지만 여기서는 특정 변수를 메모리 주소로 변환해주는 함수를 해시함수라고 한다는 것만 기억하자.

### Chaining

이미 눈치챘을 수도 있겠지만 위 나누기 방식은 한계가 있다. 만약 313호를 나누기 방식에 적용하면 213호와 마찬가지로 13이라는 메모리 주소가 출력되므로 데이터를 덮어쓰게 되는 `충돌`이 발생한다. 이러한 현상을 방지하기 위해 체이닝(Chaining) 방식을 각 메모리에 적용한다. 이는 하나의 메모리에 여러 데이터를 담을 수 있도록 하는 방법 중 하나이다.

대표적인 체이닝 방식이 `링크드리스트`를 각 메모리에 적용하는 것이다. 위에서 설명을 생략했지만 주로 더블리 링크드리스트를 적용한다. 이는 next뿐 아니라 before 변수도 추가하여 해당 Node의 앞 뒤 node의 메모리를 기록하는 방식을 말한다. 이전 링크드리스트에 대한 개념을 이해했다면 왜 메모리 주소 충돌이 발생한다 하더라도 데이터를 덮어쓰는 일 없이 하나의 메모리에 여러 데이터를 담을 수 있는 이해할 수 있을 것이다.

### Open Addressing

충돌을 해결하는 또 하나의 방식으로 이는 만약 충돌이 발생한다면 그 메모리 주소에 +1을 선형적으로 진행하면서 빈 주소를 찾을 때 까지 계속해서 탐사하는 방식을 말한다. 추가적으로 $1^2, 2^2, 3^2...$을 순차적으로 더하며 빈 주소를 찾는 제곱 탐사 방식도 있다. 어쨌든 `빈 주소를 탐사하는 개념`은 동일하다.

이러한 방식은 데이터 탐색 시 약간의 장점이 있다. 데이터 추가 시 빈 주소가 발견되면 저장하는 방식이다보니 탐색 방식도 동일하게 주소에 +@를 적용하여 찾는 방식을 취한다. 그렇기 때문에 탐색 도중 빈 주소가 발견된다면 이는 값이 저장되지 않았음을 뜻하므로 `모든 데이터를 둘러보기 전에 데이터가 없음을 확인할 수 있다.` 링크드리스트의 경우 모든 데이터를 훑어보고서야 비로소 데이터가 없음을 선언하는 구조이기에 Open Addressing의 이점이 있다.

하지만 만약 데이터를 추가할 때는 빈 주소가 아니었던 곳이 이후 데이터 삭제로 인해 빈 주소가 된다면 어떻게 될까? 데이터 탐색 시 데이터가 있음에도 없다는 오류를 발생시킬 수 있을 것이다. 이를 방지하기 위해 open addressing 에서는 데이터 삭제 시 빈 주소로 두는 것이 아니라 '데이터 없음' 과 같은 표식을 적용해 빈 주소가 되었음을 알리지만 탐색 시에는 빈 주소가 아닌 것처럼 동작하게 만들어야 한다.

### chaining과 open addressing의 시간복잡도

이 둘의 탐색, 접근, 추가, 수정, 삭제 시간복잡도는 모두 O(n)을 갖는다고 본다. 하지만 기본적인 시간 복잡도는 항상 최악의 상황을 고려한 값이라는 점을 기억해야 한다.  
링크드리스트가 적용된 Chaining 해싱 테이블은 단 하나의 메모리 주소에 담긴 링크드리스트에 모든 데이터가 순차적으로 담겼을 경우를 고려한다. 그렇기 때문에 해시 테이블임에도 $O(n)$을 언급하는 것이다. 하지만 그런 상황은 좀처럼 발생하지 않는다. 저장하고자 하는 key-value 데이터의 개수를 상회하는 메모리 공간을 확보해 둔다면 충돌 발생 자체를 줄이거나 아예 발생시키지 않을 수도 있기 때문이다. 이러한 측면에서 볼 때 충분한 메모리를 확보한다면 Chaining의 시간 복잡도는 $O(1)$로 보는 것이 맞다.  
open addressing 또한 위와 같인 관점에서 접근한다면 $O(1)$로 볼 수 있는데 자세한 설명은 아래에서 이어가겠다.

## 시간복잡도의 분할 상환 분석 적용

이 쯤에서 분할 상환 분석에 대해 집고 넘어갈 필요가 있다.  
분할 상환 분석이란, 우리가 일반적으로 적용했던 시간 복잡도를 좀 더 정확하게 계산해볼 필요가 있다는 관점에서 나온 시간 복잡도 계산 방식이다. 왜냐하면 항상 최악의 연산 속도를 고려하는 시간복잡도가 실제로 거의 발생하지 않는 자료구조가 있기 떄문이다.

분할 상환은 탐색 횟수에 따른 평균, 기대값 시간 복잡도를 구하는 방식으로 접근한다. 만약 [1, 3, 7, 4, 9]라는 배열에서 9를 찾는 탐색을 한다면 이는 최악에 해당하겠지만 과연 얼마나 9를 찾느냐는 말이다. 일반적으로 리스트의 데이터 탐색 시간 복잡도를 무조건 적으로 최악으로 취급해도 되는가?

이러한 의견에 따라 open addressing은 분할 상환 분석에서 총 메모리에 데이터가 든 비율인 'load factor'라는 수치를 활용한다. 즉 10개의 메모리에 9개의 데이터가 들어있다면 load factor는 0.9가 된다는 말이다.  
똑똑한 수학자들의 복잡한 증명을 통해 밝혀진 open addressing의 분할 상환 계산법은 $1 / (1 - \alpha)$이다. ($\alpha=load factor$)

즉 알파가 0.9라면 평균적으로 10번의 선형 탐지를 진행하게 된다는 계산이 나오는데 여기서 만약 알파를 0.5 이하로 유지하도록 메모리 관리를 한다면 그 기대값이 2가 나온다. 즉 분할 상환 분석으로 $O(2)$가 나온다는 것이다. 결론적으로 알파가 0.9가 되어도 $O(10)$에 해당하며 이는 $O(n)$으로 취급할 만큼 높은 수치가 아니므로 사실상 $O(1)$로 볼 수 있다.

대체적으로 분할 상환 분석을 적용하여도 기본 시간 복잡도와 큰 차이를 보이지 않는 경우가 대부분이지만 그렇지 않은 케이스가 있기 때문에 이러한 접근법을 사전에 알아둘 필요가 있다.

## Queue, Stack

### Queue

First In First Out(FIFO) 방식으로 데이터를 처리하는 추상 자료형을 말한다. 한국말로 '대기열'이라고 부르며 말 그대로 줄을 서는 것과 같은 데이터 처리 방식이다. 먼저 줄을 선 사람이 먼저 입장(대기열에서 퇴장)하는 것처럼 task를 줄 세운 뒤 하나씩 꺼내서 처리하기 위한 자료형이다.  
이러한 방식은 더블리 링크드 리스트 자료 구조를 적용하는 것이 유리하다. 왜냐하면 리스트의 가장 앞 단의 데이터를 pop하고, 끝 단에 데이터를 추가하는 방식은 더블리 링크드 리스트의 시간 복잡도가 $O(1)$이기 떄문이다.

### Stack

Last In First Out(LIFO) 방식으로 접시를 쌓아놓은 그림을 상상하면 이해하기 쉽다. 가장 마지막에 저장한 데이터가 가장 먼저 삭제되는 방식으로 이 역시 더블리 링크드 리스트가 유리하지만 사실상 동적 배열도 분환 상환 분석으로 접근했을 떄 $O(1)$이 나오므로 사용 가능하다.

#### Stack에서 동적 배열이 링크드 리스트와 맞먹는 이유

배열은 맨 뒤에 데이터가 추가될 때 메모리를 확장할 가능성이 있다. 가령 [1, 2, 3] 배열에서 4를 추가하는 순간 8개 요소를 담을 배열을 재생성, 복붙하는 작업을 추가로 진행하기 때문이다. 이는 $O(n)$에 해당한다. 하지만 메모리를 확장하는 단위가 곱하기 2라는 점에 주목해야 한다. 예시 대로라면 16, 32, 64, 128, 256...과 같은 순서로 확장해 나갈 것이다. 그렇다면 데이터가 3000개인 상황에서 생성된 배열의 크기는 4096일 텐데 앞으로 1096개의 데이터를 append할 동안은 시간복잡도가 $O(1)$다. 데이터가 더 많을 수록 더더욱 $O(n)$이 발생할 횟수는 미미해져 갈 것이고, 평균적으로 $O(1)$에 근사하게 될 것이다. 그래서 배열의 맨뒤 데이터 추가 연산은 분할 상환 분석에 의해 O(1)로 취급한다.