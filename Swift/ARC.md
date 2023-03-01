# ARC(Automatic Reference Counting)

- 객체의 수명은 초기화 함수 init()에 의해 시작되고 마지막 사용 이후 끝이 난다.

- ARC는 객체의 수명이 끝난 후 메모리로부터 할당을 해제한다.

- ARC는 객체의 수명을 reference count를 통해 추적한다.

- retain 혹은 release를 삽입하는 작업을 하는 스위프트 컴파일러에 의해 작동합니다. 

- runtime 때, retain은 reference count를 증가시키고 release는 reference count를 감소시킴

- reference count가 0이 되면 객체는 메모리에서 해제(deallocate)된다.

  

  <br/>

  

### 컴파일 타임에 retain/release가 삽입되는 과정

```swift
class Traveler {
  var name: String
  var destination: String?
}

func test() {
  let traveler1 = Traveler(name: "Lilly")
  
  // <2> retain (바로 아래 traveler2에 대한 삽입)
  let traveler2 = traveler1
  // <1> release(traveler1)
  
  traveler2. destination = "Big Sur"
  // <3> release(traveler2)
  
  print("Done traveling")
}
```

1. <1>은 traveler1의 마지막 사용 이후 컴파일러에 의해 release가 삽입됩니다.
2. traveler1은 init()함수에 의해 인스턴스가 생성될 때 reference count가 1로 설정되었기에 따로 retain을 삽입하지 않습니다.
3. 이후 <2>처럼 Traveler에 대한 새로운 참조가 생기므로 retain이 삽입되고, <3>처럼 traveler2가 마지막으로 사용된 후 release가 삽입됩니다.



<br/>



### 런타임에 일어나는 일

1. Traveler객체가 힙에 생성되고 reference count는1로 초기화 됩니다.
2. 새로운 참조를 준비하며 retain operation이 실행됩니다. 그리고는 곧 reference count가 2로 증가합니다.
3. 이제 traveler2는 Traveler객체를 참조합니다.
4. traveler1의 마지막 사용 이후 release operation이 실행되고 reference count가 1개로 감소합니다.
5. Traveler객체의 destination이 Big Sur로 업데이트 됩니다.
6. traveler2의 마지막 사용이 끝난 후 release operation이 실행되고 reference count가 0으로 감소합니다.
7. 메모리에서 할당이 해제됩니다.



<br/>

### 객체의 수명

<img width="526" alt="CleanShot 2023-01-04 at 11 37 42@2x" src="https://user-images.githubusercontent.com/42647277/210495119-d9c21881-c274-4fb4-9f2b-c780c390af29.png">

- 객체의 수명은 사용 기반(use-based)입니다.
- 객체에게 보장된 최소한의 수명은 초기화부터 마지막 사용까지 입니다.
- 중괄호가 끝날 때까지 객체의 수명이 보장되는 C++언어와는 다릅니다.

<br/>

<img width="323" alt="CleanShot 2023-01-04 at 11 58 10@2x" src="https://user-images.githubusercontent.com/42647277/210495224-34fc8bb1-aed8-4428-87d0-8f1e9a5a0987.png">

- 예제에서는 객체의 마지막 사용 사용 이후 즉시 메모리에서 할당이 해제되었습니다

<br/>

<img width="547" alt="CleanShot 2023-01-04 at 12 01 10@2x" src="https://user-images.githubusercontent.com/42647277/210495268-124db1fa-9140-4124-aae8-c2b43a71c50d.png">

- 하지만 실제로 객체의 수명은 스위프트의 컴파일러에 의해 삽입된 retain/release 구문에 의해 결정되었습니다. 
- ARC 최적화에 따라 객체의 수명은 최소한의 보장과는 다를 수 있습니다. 즉, 마지막 사용을 넘어서까지 수명이 길어질 수 있다는 것입니다.

<br/>

<img width="305" alt="CleanShot 2023-01-04 at 12 08 46@2x" src="https://user-images.githubusercontent.com/42647277/210495366-cdd4cfdd-e11d-4352-8895-8fe460cc2182.png">

- 그런 경우에는 마지막 사용을 넘어 위와같이 메모리에서 해제될 수 있습니다.

<br/>

- 대부분의 경우 객체의 정확한 수명은 중요하지 않습니다.
- weak, unowned, deinitializer과 같은 스위프트의 기능을 사용한다면 객체의 수명을 관찰할 수 있습니다.
- guaranteed object(마지막 사용 이후 메모리에서 해제되는 일반적인 객체)가 아닌 observed object(마지막 사용 이후 즉시 메모리에서 해제되지 않고 좀 더 오래 존재하는 객체)에 의존한다면 이후 문제가 발생할 수 있습니다.


<br/> <br/>

### Strong Reference cycle

- 위 Traveler객체에다가 코드를 좀 더 추가시켜 reference cycle에 대해 살펴봅시다.

```swift
class Traveler {
  var name: String
  var account: Account?
  func printSummary() {
    if let account = account {
      print("\(name) has \(account.points) points")
    }
  }
}

class Account {
  var traveler: Traveler
  var points: Int
}

func test() {
  let traveler = Traveler(name: "Lily")
  let account = Account(traveler: traveler, points: 1000)
  traveler.account = account
  traveler.printSummary()
}
```



<img width="566" alt="CleanShot 2023-01-04 at 12 48 47@2x" src="https://user-images.githubusercontent.com/42647277/210495437-58f6c0c5-900e-4278-a9b1-332d2603de8c.png">

- Traveler객체는 reference count가 1로 되면서 힙에 생성되었습니다.

<br/>

<img width="582" alt="CleanShot 2023-01-04 at 12 56 11@2x" src="https://user-images.githubusercontent.com/42647277/210495496-043eb749-bc2b-41e4-a628-2e6a00385940.png">

- 그리고 Account객체는 reference count가 1로 되면서 힙에 생성되었습니다.
- Account객체가 Traveler객체를 참조함에따라 Traveler의 reference count가 2로 증가하였습니다.

<br/>

<img width="580" alt="CleanShot 2023-01-04 at 12 59 18@2x" src="https://user-images.githubusercontent.com/42647277/210495546-3e04afa2-ba92-49fc-a701-7eb1a9e7af05.png">

- Traveler가 Account를 참조함에따라 Account의 reference count가 2로 증가하였습니다.
- Account를 참조하는 것이 마지막 사용입니다.

<br/>

<img width="572" alt="CleanShot 2023-01-04 at 13 01 54@2x" src="https://user-images.githubusercontent.com/42647277/210495594-ef04fc8b-2bb2-4cde-9e21-f1fa15682395.png">

- 그리고는 Account의 reference count는 1로 감소합니다.


<img width="579" alt="CleanShot 2023-01-04 at 13 02 36@2x" src="https://user-images.githubusercontent.com/42647277/210495636-92299de6-7d3b-4f2a-a5c4-2cf9b33ee79b.png">

- traveler의 printSummary()가 호출됩니다. 이것이 Traveler의 마지막 사용이므로 이내 Traveler의 reference count가 1로 줄어듭니다.
- 객체의 사용이 모두 끝났는데에도 불구하고 객체들의 reference count는 여전히 1입니다. 그 이유는 reference cycle 때문입니다.
- 결과적으로 이제 객체는 절대 메모리에서 해제되지 않아 메모리 누수(memory leak)가 발생합니다.

<br/>

- weak 혹은 unowned를 사용하여 reference cycle을 깨야합니다.
- 스위프트 런타임에서는 weak으로 참조한 것에는 nil로, unowned으로 참조한 것에는 traps로 전환합니다.
- weak을 사용할지 unowned를 사용할지는 구현에 따라 다릅니다.



### weak

- If a weak reference is used to access an object while its guaranteed object lifetime has ended, and you are relying on observed object lifetime for the object to be available, you can end up with bugs in the future when the observed object lifetime changes for unrelated reasons. (정확하게 해석이 잘 안됨...ㅠㅠ)

```swift
class Traveler {
  var name: String
  var account: Account?
  func printSummary() {
    if let account = account {
      print("\(name) has \(account.points) points")
    }
  }
}

class Account {
  weak var traveler: Traveler
  var points: Int
}

func test() {
  let traveler = Traveler(name: "Lily")
  let account = Account(traveler: traveler, points: 1000)
  traveler.account = account
  traveler.printSummary()
}
```

- Account의 traveler를 weak으로 선언해줍니다.

<img width="579" alt="CleanShot 2023-01-04 at 13 22 12@2x" src="https://user-images.githubusercontent.com/42647277/210495699-9fbfcf1e-a9c0-4fea-9780-a0ee066e6377.png">

- traveler의 함수가 마지막으로 호출된 후 Traveler의 reference count는 0으로 줄어듭니다. 애초에 weak으로 선언했기 때문에 이전 과정과는 달리 reference count가 2까지 증가하지 않았죠.

<br/>

<img width="564" alt="CleanShot 2023-01-04 at 14 15 32@2x" src="https://user-images.githubusercontent.com/42647277/210495735-d178f4be-75c8-450b-8368-4b0a004908be.png">

- reference count가 0이되어 Traveler의 객체가 메모리에서 해제됩니다.

<br/>

<img width="563" alt="CleanShot 2023-01-04 at 14 17 23@2x" src="https://user-images.githubusercontent.com/42647277/210495784-632309ba-37e0-4533-8867-740d38efbefd.png">

- Traveler의 account를 통해 Account객체를 참조하고 있었지만 Traveler가 해제되었으므로 Account의 reference count도 0으로 줄어듭니다.

<br/>

<img width="560" alt="CleanShot 2023-01-04 at 14 15 50@2x" src="https://user-images.githubusercontent.com/42647277/210495838-a667a30d-9d18-4600-81b9-36f5728dcd82.png">

- 그리고는 메모리에서 해제됩니다.


- 이 예제에서는 weak을 오직 reference cycle을 해제하기 위해 사용되었습니다.

<br/> <br/>

- Traveler의 printSummary()메서드를 Account로 옮겨 살펴보겠습니다.

```swift
class Traveler {
  var name: String
  var account: Account?
}

class Account {
  weak var traveler: Traveler
  var points: Int
  func printSummary() {
    print("\(traveler!.name) has \(points) points")
  }
}

func test() {
  let traveler = Traveler(name: "Lily")
  let account = Account(traveler: traveler, points: 1000)
  traveler.account = account
  account.printSummary()
}
```



<img width="573" alt="CleanShot 2023-01-04 at 14 28 52@2x" src="https://user-images.githubusercontent.com/42647277/210495891-92876430-5aa4-404f-b5d2-20b1af96260f.png">

- traveler의 마지막 사용 이후 컴파일러가 release를 즉시 삽입한다면 Traveler의 reference count는 0으로 감소하게 됩니다.

<br/>

<img width="565" alt="CleanShot 2023-01-04 at 14 29 19@2x" src="https://user-images.githubusercontent.com/42647277/210495941-4e7f99e2-19c7-491e-96e5-7b1106c6b149.png">

- Account의 weak을 통해 참조했던 traveler는 nil이 되고

<br/>

<img width="556" alt="CleanShot 2023-01-04 at 14 29 32@2x" src="https://user-images.githubusercontent.com/42647277/210495984-9398784f-c2b6-4b1b-a940-2505dca72fc5.png">

- Traveler객체는 메모리에서 해제됩니다.

<br/>

<img width="498" alt="CleanShot 2023-01-04 at 14 29 39@2x" src="https://user-images.githubusercontent.com/42647277/210496017-ed3def00-75dc-43a7-83f1-927e32f0badf.png">

- 그리고 printSummary() 함수가 호출되면 강제 언래핑했던 부분에서 크래쉬가 발생합니다. Account가 참조하던 traveler는 이미 메모리에서 해제되었거든요.

<br/>

<img width="464" alt="CleanShot 2023-01-04 at 14 29 46@2x" src="https://user-images.githubusercontent.com/42647277/210496061-e2f0f15f-c344-4752-8a8d-42a05a0304cc.png">

- 옵셔널 바인딩을 통해 크래쉬를 예방할 수 있습니다.
- 하지만 여기서 옵셔널 바인딩은 사실 문제를 더 악화시킵니다.
- 명백한 크러쉬가 없으면 조용하게 숨겨진 버그로 남기 때문입니다.

<br/>  

- weak과 unowned를 이용한 참조를 안전하게 다루는 몇가지 기술이 있습니다.
- 각 기술마다 초기 구현 비용(upfront implement cost)과 유지 관리 비용(continuous maintenance)이 다릅니다.

<br/>



## Safe techniques in Weak and unowned references

- 스위프트는 `withExtendedLifetime()`메서드를 통해 객체의 수명을 명시적으로 연장할 수 있는 유틸리티를 제공합니다.



<img width="558" alt="CleanShot 2023-01-04 at 15 59 07@2x" src="https://user-images.githubusercontent.com/42647277/210500842-2d978182-7f47-45a0-8fc0-26e09c23a9c6.png">

- `withExtendedLifetime()`를 사용하면 printSummary()함수가 호출되는 동안 Traveler 객체의 수명을 안전하게 연장하여 잠재적인 버그의 발생을 예방할 수 있습니다.



<br/>



<img width="558" alt="CleanShot 2023-01-04 at 16 01 55@2x" src="https://user-images.githubusercontent.com/42647277/210501163-f1949747-d7f4-4029-b21b-1e9157cf7a35.png">

- test()함수 스코프의 맨 마지막에 클로저 내부를 빈 상태로 구현하여도 동일한 효과를 얻을 수 있습니다. 

<br/>



<img width="558" alt="CleanShot 2023-01-04 at 16 03 55@2x" src="https://user-images.githubusercontent.com/42647277/210501440-9628744e-858b-40f4-8889-9e0adabda2a4.png">

- 더 복잡한 경우에, defer를 사용하여 객체의 수명을 test()함수 범위 끝까지로 확장하도록 컴파일에 요청할 수도 있습니다.

<br/>



- `withExtendedLifeTime()`은 객체의 수명에 관한 버그를 쉽게 해결할 수 있어 보이지만 사실 취약한 점이 있습니다.
- 그건 바로 사용자에게 정확히 사용해야 할 책임이 있다는 것입니다.
- weak의 사용이 버그를 일으킬 가능성이 있을 때마다 `withExtendedLifeTime()` 을 사용하여 확인해야 합니다.
- 여차하면 코드베이스 전체 범위에서 자주 사용되며 이에따라 유지 관리에 대한 비용이 커지게 됩니다.



<br/>



<img width="554" alt="CleanShot 2023-01-04 at 16 12 32@2x" src="https://user-images.githubusercontent.com/42647277/210502606-3331fd6f-3c32-4b68-9a2b-1a69c3b9b1fb.png">

- 더 나은 API가 되도록 class를 재정의하는 것이 훨씬 원칙적인 접근 방법입니다.
- 객체에 대한 접근을 오직 strong reference로만 제한하는 것이 예상치 못한 일을 방지할 수 있습니다.
- weak으로 선언되어있는 traveler에 대해 private으로 감췄습니다.
- 이제 test()함수는 strong reference를 통해 printSummary()를 호출하여 잠재적인 버그를 제거합니다.



<br/>



<img width="556" alt="CleanShot 2023-01-04 at 16 30 42@2x" src="https://user-images.githubusercontent.com/42647277/210505149-9632e5e8-077d-45cc-9838-ea66e646a605.png">

- 클래스 설계에 주의하지 않으면, weak과 unowned의 사용은 성능에 대한 비용을 요구할 뿐만 아니라 버그에도 노출될 수 있습니다.
- weak과 unowned가 왜 필요한지 생각해야 합니다. 단순히 순환 참조를 피하기 위해서? 단지 피하기 위해서라면 순환 관계에 놓여있는 class들을 트리 구조로 변경하여 피할 수 있습니다.
- 사실 이 예제에서도 Traveler는 Account클래스가 필요했지만 그 반대는 필요하지 않습니다. Account는 단지 traveler의 개인 정보가 필요한 것 뿐입니다.
- 위와 같이 PersonalInfo를 새로 만든다면 순환 참조를 피할 수 있습니다.
- week과 unowned가 필요하지 않도록 설계하는 것은 구현에 대한 비용이 들 수 있겠으나, 이것은 잠재적으로 존재할 버그를 없애기 위한 가장 확실한 방법입니다.



----

## 출처

- https://developer.apple.com/videos/play/wwdc2021/10216/
- https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html
