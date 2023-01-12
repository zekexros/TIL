# Deque

- 양쪽 끝에서 삽입과 삭제가 모두 가능한 자료 구조의 한 형태입니다.

```swift
public struct Deque<T> {
  private var array = [T]()

  public var isEmpty: Bool {
    return array.isEmpty
  }

  public var count: Int {
    return array.count
  }

  public mutating func enqueue(_ element: T) {
    array.append(element)
  }

  public mutating func enqueueFront(_ element: T) {
    array.insert(element, at: 0)
  }

  public mutating func dequeue() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.removeFirst()
    }
  }

  public mutating func dequeueBack() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.removeLast()
    }
  }

  public func peekFront() -> T? {
    return array.first
  }

  public func peekBack() -> T? {
    return array.last
  }
}
```

- 배열을 이용하여 구현하였습니다. equeue와 dequeue는 배열의 앞과 뒤에서 모두 가능합니다.

<br/>

### 예시

```swift
var deque = Deque<Int>()
deque.enqueue(1)
deque.enqueue(2)
deque.enqueue(3)
deque.enqueue(4)

deque.dequeue()       // 1
deque.dequeueBack()   // 4

deque.enqueueFront(5)
deque.dequeue()       // 5
```

- `Deque`는 간단하지만 효율적이진 않습니다. `enqueueFront()`와 `dqeueue()` 작업의 시간복잡도는 O(n)입니다.
- `Deque`의 동작사양에만 맞춘 구현으로 좀 더 효율적인 방법으로의 구현을 아래 소개해 드리겠습니다.

<br/>

### Deque의 원리와 시간복잡도

-  `enqueueFront()`와 `dqeueue()`는 배열의 앞에서 작업하므로 O(n)의 시간복잡도를 가졌습니다.

- 배열의 앞에 있는 요소를 삭제하면 남아있는 모든 요소들은 메모리에서 이동해야 합니다.

  1. 1~4까지 숫자가 담긴 배열이 있다고 가정할 때

     ```swift
     [1, 2, 3, 4]
     ```

  2. `dequeue()`는 배열안의 1을 제거하고 남아있는 2, 3, 4는 앞으로 이동합니다.

     ```swift
     [ 2, 3, 4 ]
     ```

     - 배열의 모든 요소가 하나씩 앞으로 이동해야 하므로 O(n) 작업입니다. 

  3. 마찬가지로 배열의 앞부분에 요소를 삽입하는 것도 남아있는 다른 요소들을 한칸 씩 뒤로 보내야하기에 O(n) 작업입니다. 

     ```swift
     [ 5, 2, 3, 4 ]
     ```

- `enqueue()`와 `dequeueBack()`에서 문제가 되지 않는 이유는 배열의 끝에서 수행되기 때문입니다.

- Swift에서 배열은 크기 조정 가능한 형태로 구현되었기 때문에 배열 뒷부분에 일정량의 여유 공간을 확보해둡니다.

- 앞서 첫 번째 단계에 있는 1~4까지 숫자가 담긴 배열을 생성할 때 배열은 메모리에서 아래와 같이 생겼습니다.

  ```swift
  [ 1, 2, 3, 4, x, x, x ]
  ```

- 여기서 x는 아직 요소가 들어오지 않고 공간만 확보한 모습을 나타냅니다. `enqueue(6)`을 호출하면 사용되지 않은 자리(x)에 복사됩니다.

  ```swift
  [ 1, 2, 3, 4, 6, x, x ]
  ```

- 배열의 시작 부분에 위와 동일한 트릭을 사용하여 효율적인 `Deque`를 만들어보려고 합니다.

  ```swift
  [ x, x, x, 1, 2, 3, 4, x, x, x ]
  ```

  

### 새로운 Deque

```swift
public struct Deque<T> {
  private var array: [T?]
  private var head: Int
  private var capacity: Int
  private let originalCapacity:Int

  public init(_ capacity: Int = 10) {
    self.capacity = max(capacity, 1)
    originalCapacity = self.capacity
    array = [T?](repeating: nil, count: capacity)
    head = capacity
  }

  public var isEmpty: Bool {
    return count == 0
  }

  public var count: Int {
    return array.count - head
  }

  public mutating func enqueue(_ element: T) {
    array.append(element)
  }

  public mutating func enqueueFront(element: T) {
    if head == 0 {
      capacity *= 2
      let emptySpace = [T?](repeating: nil, count: capacity)
      array.insert(contentsOf: emptySpace, at: 0)
      head = capacity
    }

    head -= 1
    array[head] = element
  }
  
  public mutating func dequeue() -> T? {
    guard head < array.count, let element = array[head] else { return nil }

    array[head] = nil
    head += 1

    if capacity >= originalCapacity && head >= capacity*2 {
      let amountToRemove = capacity + capacity/2
      array.removeFirst(amountToRemove)
      head -= amountToRemove
      capacity /= 2
    }
    return element
  }

  public mutating func dequeueBack() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.removeLast()
    }
  }

  public func peekFront() -> T? {
    if isEmpty {
      return nil
    } else {
      return array[head]
    }
  }

  public func peekBack() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.last!
    }
  }  
}
```

- 바뀐 점은 제네릭 타입 T가 옵셔널로 변경되었다는 것. 이유는 배열의 요소를 빈 형태로 바꿔줘야 하기 때문입니다.

- init 메소드를 살펴보면 nil값을 포함하는 새 배열을 할당합니다. 여유 공간을 확보하는 작업으로 일반적으로 10개의 빈자리를 확보합니다.

- `head`는 객체 배열의 맨 앞을 가리키는 인덱스입니다. 대기열이 비어있기 때문에 head는 배열의 끝을 넘어서는 인덱스를 가리킵니다.

  ```swift
    [ x, x, x, x, x, x, x, x, x, x ]
                                     |
                                     head
  ```

- `enqueueFront(5)`메소드를 실행하면 head는 한칸 앞으로 이동하고 새로운 객체를 head가 있는 자리로 복사합니다.

  ```swift
  [ x, x, x, x, x, x, x, x, x, 5 ]
                               |
                               head
  ```

- `enqueueFront(7)`을 실행하면 다음과 같이 됩니다.

  ```swift
  [ x, x, x, x, x, x, x, x, 7, 5 ]
                            |
                            head
  ```

- `enqueueFront()`는 그저 값을 배열안에 복사하는게 끝이므로 시간복잡도 O(1) 고정입니다.

<br/>

- 뒷부분에 요소를 넣는 것은 별다른 변화가 없습니다. 예를 들면 `enqueue(1)` 메소드를 실행할 때 말이죠.

  ```swift
  [ x, x, x, x, x, x, x, x, 7, 5, 1, x, x, x, x, x, x, x, x, x ]
                            |
                            head
  ```

- 살펴볼 점은 배열의 사이즈가 바뀌었다는 것입니다. 1을 넣을 때 공간이 없었습니다. 그래서 Swift는 배열의 크기를 늘리기로 결정하고 많은 빈자리들을 배열의 뒤에 추가하였습니다. 이 상태에서 `enqueue(2)`를 실행하면 빈자리에 들어갈 것입니다.

  ```swift
  [ x, x, x, x, x, x, x, x, 7, 5, 1, 2, x, x, x, x, x, x, x, x ]
                            |
                            head
  ```

  > 이 때, 뒤에 있는 빈자리들은 print문으로 출력해도 보이지 않습니다. 스위프트는 우리가 헷갈릴까 봐 안보이도록 구현하였습니다. 우리가 직접 넣은 nil만 볼 수 있습니다.

<br/>

- `dequeue()`메소드는 `enqueueFront()`메소드의 반대입니다. head에 위치한 값을 읽고 nil로 바꿈으로서 요소를 제거합니다. 그리고 head의 위치를 한 칸 오른쪽으로 이동합니다.

  ```swift
    public mutating func dequeue() -> T? {
      guard head < array.count, let element = array[head] else { return nil }
  
      array[head] = nil
      head += 1
  
      return element
    }
  ```

- 여기서 한가지 문제가 있습니다. 만약 많은 요소들을 배열의 앞쪽에 추가한다면, 남아있는 빈 배열이 없을 떄가 생깁니다. 이게 배열의 뒷부분이라면 스위프트가 알아서 resize해주겠지만.. 앞은 resize를 구현해야 합니다.

  ```swift
    public mutating func enqueueFront(element: T) {
      if head == 0 {
        capacity *= 2
        let emptySpace = [T?](repeating: nil, count: capacity)
        array.insert(contentsOf: emptySpace, at: 0)
        head = capacity
      }
  
      head -= 1
      array[head] = element
    }
  ```

- head가 0이되면 앞에 남아있는 공간이 없을 것입니다. 새로운 nil을 많이 생성해서 다시 공간을 확보해줍시다.

- 이 작업의 시간복잡도는 O(n)이지만, 공간이 없을 때만 트리거되는 작업이기에 평균은 여전히 O(1)입니다.

- 또한 앞에 남은 공간이 없어 resize될 때마다 capacity에 2가 곱해지므로 오래걸리는 위의 작업이 이전보다 점점 덜 발생합니다. 이 작업은 배열의 뒤에서는 스위프트가 이미 작업하고 있는 부분입니다.

<br/>

- 만약 많은 요소를 배열의 뒤에 추가하고 앞쪽에서 요소들을 삭제하다 보면 아래와 같은 형태를 보이게 될 것입니다.

  ```swift
  [ x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, 1, 2, 3 ]
                                                                |
                                                                head
  ```

- 배열의 앞에 있는 빈자리들은 `enqueueFront()`메소드를 사용할 때만 사용됩니다. 그러나 앞에서 요소를 넣는 작업들을 거의 안한다면 공간을 낭비하게 되겠죠. 이러한 불필요한 메모리를 낭비하지 않도록 `dequeue()`메소드를 손봐야 합니다.

<br/>

```swift
  public mutating func dequeue() -> T? {
    guard head < array.count, let element = array[head] else { return nil }

    array[head] = nil
    head += 1

    if capacity >= originalCapacity && head >= capacity*2 {
      let amountToRemove = capacity + capacity/2
      array.removeFirst(amountToRemove)
      head -= amountToRemove
      capacity /= 2
    }
    return element
  }
```

- capacity는 배열의 맨앞에 있는 빈자리의 수 입니다. head의 위치가 capacity의 2배를 넘는다면 빈공간들을 1/4로 줄일겁니다.
- 이 때, deque는 capacity와 originalCapacity와의 비교를 통해 최소한의 originalCapacity로 공간을 남겨줄 겁니다.
- 작업 전

  ```swift
  [ x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, 1, 2, 3 ]
                                  |                             |
                                  capacity                      head
  ```

- 작업후

  ```swift
  [ x, x, x, x, x, 1, 2, 3 ]
                   |
                   head
                   capacity
  ```

  

- 이렇게 메모리를 효율적으로 관리하면서 시간복잡도를 크게 향상시키는 Deque가 완성되었습니다.
- 매우 작은 배열에서는 메모리 절약 작업을 따로 하지 않습니다. 절약되는 메모리가 고작 몇바이트밖에 안되기 때문입니다.



## 참고

https://github.com/kodecocodes/swift-algorithm-club/tree/master/Deque