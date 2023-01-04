# ARC(Automatic Reference Counting)

- 객체의 수명은 초기화 함수 ini()에 의해 시작되고 마지막 사용 이후 끝이 난다.

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

- ![CleanShot 2023-01-04 at 11.37.42@2x](/Users/zeke/Desktop/CleanShot 2023-01-04 at 11.37.42@2x.png)
  - 객체의 수명은 사용 기반(use-based)입니다.
  - 객체에게 보장된 최소한의 수명은 초기화부터 마지막 사용까지 입니다.
  - 중괄호가 끝날 때까지 객체의 수명이 보장되는 C++언어와는 다릅니다.
- ![CleanShot 2023-01-04 at 11.37.42@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_bi8klZ3cfM/CleanShot 2023-01-04 at 11.58.10@2x.png)
  - 예제에서는 객체의 마지막 사용 사용 이후 즉시 메모리에서 할당이 해제되었습니다
- ![CleanShot 2023-01-04 at 12.01.10@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_C7DeI7CMKK/CleanShot 2023-01-04 at 12.01.10@2x.png)
  - 하지만 실제로 객체의 수명은 스위프트의 컴파일러에 의해 삽입된 retain/release 구문에 의해 결정되었습니다. 
  - ARC 최적화에 따라 객체의 수명은 최소한의 보장과는 다를 수 있습니다. 즉, 마지막 사용을 넘어서까지 수명이 길어질 수 있다는 것입니다.
- ![CleanShot 2023-01-04 at 12.08.46@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_3ClnI8VQrV/CleanShot 2023-01-04 at 12.08.46@2x.png)
  - 그런 경우에는 마지막 사용을 넘어 위와같이 메모리에서 해제될 수 있습니다.
- 대부분의 경우 객체의 정확한 수명은 중요하지 않습니다.
- weak, unowned, deinitializer과 같은 스위프트의 기능을 사용한다면 객체의 수명을 관찰할 수 있습니다.
- guaranteed object(마지막 사용 이후 메모리에서 해제되는 일반적인 객체)가 아닌 observed object(마지막 사용 이후 즉시 메모리에서 해제되지 않고 좀 더 오래 존재하는 객체)에 의존한다면 이후 문제가 발생할 수 있습니다.



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



- ![CleanShot 2023-01-04 at 12.48.47@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_76vdPRkjjp/CleanShot 2023-01-04 at 12.48.47@2x.png)
  - Traveler객체는 reference count가 1로 되면서 힙에 생성되었습니다.



- ![CleanShot 2023-01-04 at 12.56.11@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_oIaaqiqNoV/CleanShot 2023-01-04 at 12.56.11@2x.png)

  - 그리고 Account객체는 reference count가 1로 되면서 힙에 생성되었습니다.
  - Account객체가 Traveler객체를 참조함에따라 Traveler의 reference count가 2로 증가하였습니다.

  

- ![CleanShot 2023-01-04 at 12.59.18@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_NPikF4PLgc/CleanShot 2023-01-04 at 12.59.18@2x.png)
  - Traveler가 Account를 참조함에따라 Account의 reference count가 2로 증가하였습니다.
  - Account를 참조하는 것이 마지막 사용입니다.
- ![CleanShot 2023-01-04 at 13.01.54@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_KZBUxvwHGK/CleanShot 2023-01-04 at 13.01.54@2x.png)
  - 그리고는 Account의 reference count는 1로 감소합니다.
- ![CleanShot 2023-01-04 at 13.02.36@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_szkGrVGI0V/CleanShot 2023-01-04 at 13.02.36@2x.png)
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

- ![CleanShot 2023-01-04 at 13.22.12@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_OVm9BYRyuo/CleanShot 2023-01-04 at 13.22.12@2x.png)
  - traveler의 함수가 마지막으로 호출된 후 Traveler의 reference count는 0으로 줄어듭니다. 애초에 weak으로 선언했기 때문에 이전 과정과는 달리 reference count가 2까지 증가하지 않았죠.
- ![CleanShot 2023-01-04 at 14.15.32@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_17LIdHCOO2/CleanShot 2023-01-04 at 14.15.32@2x.png)
  - reference count가 0이되어 Traveler의 객체가 메모리에서 해제됩니다.
- ![CleanShot 2023-01-04 at 14.17.23@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_7osnVNddYr/CleanShot 2023-01-04 at 14.17.23@2x.png)
  - Traveler의 account를 통해 Account객체를 참조하고 있었지만 Traveler가 해제되었으므로 Account의 reference count도 0으로 줄어듭니다.
- ![CleanShot 2023-01-04 at 14.15.50@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_Odicd69RI8/CleanShot 2023-01-04 at 14.15.50@2x.png)
  - 그리고는 메모리에서 해제됩니다.
- 이 예제에서는 weak을 오직 reference cycle을 해제하기 위해 사용되었습니다.



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



- ![CleanShot 2023-01-04 at 14.28.52@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_tPV0OCgEci/CleanShot 2023-01-04 at 14.28.52@2x.png)
  - traveler의 마지막 사용 이후 컴파일러가 release를 즉시 삽입한다면 Traveler의 reference count는 0으로 감소하게 됩니다.



- ![CleanShot 2023-01-04 at 14.29.19@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_5Fdnj7c2XU/CleanShot 2023-01-04 at 14.29.19@2x.png)

  - Account의 weak을 통해 참조했던 traveler는 nil이 되고

    

- ![CleanShot 2023-01-04 at 14.29.32@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_P9QyPVI2LJ/CleanShot 2023-01-04 at 14.29.32@2x.png)

  - Traveler객체는 메모리에서 해제됩니다.

  

- ![CleanShot 2023-01-04 at 14.29.39@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_cegozzmePh/CleanShot 2023-01-04 at 14.29.39@2x.png)
  - 그리고 printSummary() 함수가 호출되면 강제 언래핑했던 부분에서 크래쉬가 발생합니다. Account가 참조하던 traveler는 이미 메모리에서 해제되었거든요.



- ![CleanShot 2023-01-04 at 14.29.46@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_TT5mjaEd5z/CleanShot 2023-01-04 at 14.29.46@2x.png)

  - 옵셔널 바인딩을 통해 크래쉬를 예방할 수 있습니다.
  - 하지만 여기서 옵셔널 바인딩은 사실 문제를 더 악화시킵니다.
  - 명백한 크러쉬가 없으면 조용하게 숨겨진 버그로 남기 때문입니다.

  

- weak과 unowned를 이용한 참조를 안전하게 다루는 몇가지 기술이 있습니다.
- 각 기술마다 초기 구현 비용(upfront implement cost)과 유지 관리 비용(continuous maintenance)이 다릅니다.



이어지는 내용은 2편에 이어서...



----

## 출처

- https://developer.apple.com/videos/play/wwdc2021/10216/
- https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html