# Queue

![image](https://user-images.githubusercontent.com/42647277/210544711-369c4e3b-cd49-414b-a7ae-8f8d89f0b8fa.png "Queue Image from geeksforgeeks.org")

- 큐는 스택과 유사한 데이터 구조입니다.
- 하지만 스택과는 달리 큐는 양쪽 끝이 열려 있습니다.
- 한쪽은 항상 데이터를 삽입(enqueue)하고 다른 한쪽은 데이터를 제거(dequeue)하는데 사용됩니다.
- 큐는 FIFO(First-In-First-Out) 방식을 따릅니다. 먼저 저장된 데이터가 먼저 제거됩니다.



<br/>



## Basic Operation

- enqueue
- dequeue
- front
- count
- isEmpty



<br/>



## Source Code

```swift
public struct Queue<T> {
  fileprivate var array = [T]()

  public var isEmpty: Bool {
    return array.isEmpty
  }
  
  public var count: Int {
    return array.count
  }

  public mutating func enqueue(_ element: T) {
    array.append(element)
  }
  
  public mutating func dequeue() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.removeFirst()
    }
  }
  
  public var front: T? {
    return array.first
  }
}
```

- 위의 작성된 Queue는 잘 작동하지만 최적화되진 않았습니다.
- enqueue()는 배열의 끝에 요소를 추가하기 때문에 배열의 크기와 상관없이 시간복잡도는 O(1) 입니다.
- 배열에 항목을 추가하는 것의 시간복잡도가 O(1)인 이유는 <u>Swift의 배열은 항상 끝에 빈 공간이 있기 때문입니다.</u>

<br/>



## 스위프트 배열의 특성과 간단하게 구현된 Queue

예시를 살펴보겠습니다.

```swift
var queue = Queue<String>()
queue.enqueue("Ada")
queue.enqueue("Steve")
queue.enqueue("Tim")
```

- 이렇게 queue에 3개의 요소를 추가한다면 배열은 아래와 같이 보일 것 입니다.

```swift
[ "Ada", "Steve", "Tim", xxx, xxx, xxx ]
```

- 여기서 `xxx`는 아직 실제 요소들로 채워지지 않은 그저 <u>공간</u>을 나타냅니다.
- 배열에 새 요소를 추가하면 `xxx`에 채워지게 됩니다.

```swift
[ "Ada", "Steve", "Tim", "Grace", xxx, xxx ]
```

- 배열의 끝에는 제한된 수의 사용되지 않은 스팟(위 예시에서 xxx)들이 존재합니다.

- 마지막 `xxx`가 사용되고 다른 항목을 추가하려는 경우 더 많은 공간을 확보하기 위해 배열의 크기를 조정해야 합니다.

- 크기를 조정하는 과정에는 <u>새로운 메모리 할당</u>과 <u>기존의 데이터를 새로운 배열로 복사하는 작업</u>이 포함됩니다. 이 과정은 O(n)의 시간복잡도를 가집니다.

- 이것은 가끔 발생하기 때문에 배열 끝에 새로운 요소를 추가하는 시간복잡도는 평균 O(1) 혹은 O(1) "amortized" 입니다.

- dequeue는 다릅니다. dequeue는 배열의 가장 앞부분을 제거합니다. 이것은 메모리에서 배열의 요소들을 이동해야 하기 때문에 항상 O(n)의 시간복잡도를 가집니다.

  ```
  before   [ "Ada", "Steve", "Tim", "Grace", xxx, xxx ]
                     /       /      /
                    /       /      /
                   /       /      /
                  /       /      /
   after   [ "Steve", "Tim", "Grace", xxx, xxx, xxx ]
  ```

- 첫 번째 요소 "Ada"를 대기열에서 빼면 <u>(1)"Ada" 대신 "Steve"</u>, <u>(2)"Steve" 대신 "Tim"</u>, <u>(3)"Tim" 대신 "Grace"</u>가 복사됩니다.

- 모든 요소를 이동하는 것은 항상 O(n) 작업입니다. 그러므로 queue를 간단하게 구현한다면 enqueue는 효율적이지만 dequeue는 그렇지 않습니다.

<br/>



##  효율적으로 구현된 Queue

- 효율적으로 dequeue하기 위한 아이디어는 요소를 배열에서 제거할 때 요소들을 앞으로 이동시키는 것이 아닌 제거된 자리를 빈자리로 표시하는 것입니다. 

- "Ada"가 dequeue된 이후 배열은 아래와 같습니다.

  ```swift
  [ xxx, "Steve", "Tim", "Grace", xxx, xxx ]
  ```

- "Steve"가 dequeue된 이후 배열은 아래와 같습니다.

  ```swift
  [ xxx, xxx, "Tim", "Grace", xxx, xxx ]
  ```

- 앞의 빈공간은 재사용되지 않으므로 남아있는 요소들을 전면으로 이동하여 배열을 주기적으로 가공 할 수 있습니다.

- 이러한 가공 과정은 시간복잡도 O(n)인 메모리 shifting(removeFirst()함수를 호출한 부분)이 포함됩니다. 아주 가끔 발생하기에 dequeue는 평균적으로는 O(1)입니다.

```swift
public struct Queue<T> {
  fileprivate var array = [T?]()
  fileprivate var head = 0
  
  public var isEmpty: Bool {
    return count == 0
  }

  public var count: Int {
    return array.count - head
  }
  
  public mutating func enqueue(_ element: T) {
    array.append(element)
  }
  
  public mutating func dequeue() -> T? {
    guard head < array.count, let element = array[head] else { return nil }

    array[head] = nil
    head += 1

    let percentage = Double(head)/Double(array.count)
    if array.count > 50 && percentage > 0.25 {
      array.removeFirst(head)
      head = 0
    }
    
    return element
  }
  
  public var front: T? {
    if isEmpty {
      return nil
    } else {
      return array[head]
    }
  }
}
```

- dequeue과정은 요소를 nil로 바꿀 뿐 배열에서 제거되지 않으므로 배열의 크기는 계속해서 커집니다.
- 따라서 배열을 주기적으로 정리해야 합니다.
- 배열이 요소가 최소 50개 이상일 때 그리고 배열의 25% 이상이 사용되지 않을 때(nil일 때) 정리합니다. 

 <br/>



- 시간복잡도 O(1)로 배열의 첫번 째 요소를 제거하는 새로운 방법을 마주하고는 굉장히 신선하다(?)라는 생각을 갖게 되었다.
- 과거에 reversed()를 이용하여 시간복잡도 O(1)로 배열의 첫번 째 요소를 제거하려는 시도를 했었지만 끝내 완성을 못하였던 기억이 있는데 위의 소개된 방법을 알고리즘 문제에 사용하며 시간복잡도를 줄여봐야겠다.



<br/>



## 출처

- https://github.com/kodecocodes/swift-algorithm-club/tree/master/Queue
- https://www.kodeco.com/848-swift-algorithm-club-swift-queue-data-structure
- https://www.tutorialspoint.com/data_structures_algorithms/dsa_queue.htm