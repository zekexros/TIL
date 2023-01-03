# Stack

## 스택이란??
- 스택이란, LIFO(Last-In-First-Out)원칙을 따르는 선형 데이터 구조입니다.
- 스택의 최상위 요소를 가리키는 하나의 포인터(top pointer)만 존재합니다.
- 새로운 요소는 스택의 맨 위에 추가되며, 반대로 삭제될 때에도 맨 위에서부터 삭제됩니다.
- 즉, 스택이란 <U>스택의 맨 위에서 삽입 및 삭제가 수행되는 컨테이너</U> 라고 정의할 수 있습니다.
- 스택이라고 불리우는 이유는 실제 우리 주변에서의 스택, 책더미 등과 같이 동작하기 때문입니다.
- 스택은 용량이 미리 정의된 추상적인 데이터 타입으로서, 크기가 제한된 요소를 저장할 수 있습니다.

<br/>

![Stack](https://user-images.githubusercontent.com/42647277/210309480-a9845842-85c8-4144-a2b1-c54e10770a5c.png)
- 요소를 최대 5개까지 저장할 수 있는 스택의 모습입니다.
- 아래서부터 1개씩 저장됩니다.
- 요소가 들어오거나 나가는 문은 오직 1개입니다.


  <br/>

## Standard Stack Operations
- push
  - 스택에 요소를 삽입하는 작업 
  - 스택이 가득차면 overflow 조건이 달성됩니다 

  <br/>
- pop
  - 스택에 요소를 제거하는 방법
  - 스택이 비어있으면 underflow상태로 한다

  <br/>
- isEmpty
  - 스택이 비어있는지 여부를 나타냄

  <br/>

- isFull
  - 스택이 다찼는지 여부를 나타냄

  <br/>

- peek
  - 스택 최상단에 있는 요소를 반환함
  - 스택의 최상단에 위치한 요소에 대해서 pop은 요소를 삭제하는 것이고 peek은 요소를 그저 반환하는 것임
  
  <br/>
  
- count
  - 스택에 있는 요소의 총 개수를 반환함

  <br/>

- change
  - (매개변수를 통해)주어진 위치에 있는 요소를 변경함

  <br/>

- display
  - 스택에 들어있는 요소를 모두 인쇄함

  <br/>

## 소스 코드
```swift
public struct Stack<T> {
  fileprivate var array = [T]()

  public var isEmpty: Bool {
    return array.isEmpty
  }

  public var count: Int {
    return array.count
  }

  public mutating func push(_ element: T) {
    array.append(element)
  }

  public mutating func pop() -> T? {
    return array.popLast()
  }

  public var peek: T? {
    return array.last
  }
  
  public func display() {
    array.forEach { print($0) }
  }
}
```

- push는 새로운 요소를 배열의 시작부분이 아닌 끝부분에 배치합니다.
- 배열의 시작부분에 요소를 삽입하는 부분은 시간복잡도가 <u>O(n)</u>일 정도로 마냥 간단한 작업은 아닙니다. 그 이유는, 기존 배열에 들어있던 요소들을 메모리 영역 안에서 한칸씩 이동해야 하기 때문입니다.
- 반대로 배열의 마지막 부분에 요소를 추가하는건 배열의 크기와 상관없이 항상 시간복잡도가 O(1) 입니다.



<br/>



### 번외

- 함수를 호출할 때마다 CPU는 스택에 반환 주소(return address)를 배치합니다. 그리고 함수가 끝날 때, CPU는 해당 주소를 사용하여 호출자에게로 돌아갑니다.
- 따라서, 재귀함수처럼 너무 많은 함수를 호출하면 CPU 스택의 공간이 부족하여 <u>stack overflow</u>가 발생합니다.

  <br/>

----
출처
- https://github.com/kodecocodes/swift-algorithm-club/tree/master/Stack
- https://www.kodeco.com/800-swift-algorithm-club-swift-stack-data-structure
- https://www.javatpoint.com/data-structure-stack