# Understanding Swift Concurrency’s AsyncStream

## Producing a simple async stream

- 비동기 스트림은 다양한 방법으로 생성될 수 있습니다. async stream을 생성하는 가장 쉬운 방법은 `AsyncStream(unfolding:)`이니셜라이저 입니다:

  <img width="900" alt="CleanShot 2023-03-03 at 14 35 10@2x" src="https://user-images.githubusercontent.com/42647277/222639830-14bb4a0c-7eec-485b-a672-79df0fd91288.png">



- 위 예제로는 특별히 유용하다고 보기 어렵지만 `AsyncStream(unfolding:)`의 개념이 얼마나 단순한지 보여줍니다.
- 비동기 스트림에 대한 return값을 생산하고 반환할 때마다 이 `AsyncStream`버전을 사용합니다.
- `unfolding`에 전달된 클로저는 비동기이므로 unfolding closure 내에서 비동기 작업을 `await`할 수 있음을 의미합니다.
- 스트림에서 값을 만들어낼 것으로 예상될 때마다 unfoled closure가 호출됩니다. 이 말은 실제로 클로저가 호출되고 작업을 실행하고 값을 반환하고 다음 클로저가 호출됨을 의미합니다.
- 이것은 for루프가 취소되고 async for루프를 포함하는 작업이 취소될 때까지, 혹은 unfolding closure에서 nil을 반환할 때까지 반복됩니다.



- 값의 스트림을 생성하는 `AsyncStream(unfolding:)`은 편리하지만 아래와 같은 상황에서 특히 유용합니다.
  - 요소를 생성하기 위해 기다려야 하는 비동기 작업을 수행하기 원할 때
  - 소유한 API를 연결할 때 back pressure를 처리해야 할 때



- Delegate를 기반으로 하는 기존 API 혹은 콜백을 활용하여 결과를 전달하는 API를 브릿징하는 경우 아마 `AsyncStream(unfolding:)`을 사용할 수 없을 것입니다.
- 이는 에러 발생 가능성이 가장 낮게 비동기 스트림을 구축하는 간단한 방법이지만 기존 코드를 스위프트의 새로운 동시성 시스템과 연결하는데 적합하지 않은 경우가 많습니다.
- `AsyncStream`용 continuation기반 API에서 더 많은 유연성을 제공합니다.

<br/>

## Producing an async stream with a continuation

- continuation을 사용하면 비동기 스트림 객체를 구성하고 값을 사용할 수 있을 때마다 비동기 스트림을 통해 값을 보낼 수 있습니다.

- `AsyncStream(build:)`이니셜라이저를 사용하여 `AsyncStream`을 생성을 통해 할 수 있죠:

  <img width="905" alt="CleanShot 2023-03-03 at 15 40 18@2x" src="https://user-images.githubusercontent.com/42647277/222649636-dadf493e-bc30-47c8-bf50-2f3830a002f2.png">

- `yield`를 호출하여 값을 생성합니다.
- 전송할 값이 있을 때마다 전송하려는 값이 있는 continuation에서 `yield`를 호출해야 합니다.



- Delegate기반 API를 래핑하는 `AsyncStream`을 구축하는 경우에도 delegate 메서드가 호출될 때마다 continuation에서 `yield`를 호출할 수 있습니다.

- 예를 들어 새로운 사용자 위치를 사용할 수 있을 때마다 `CLLocationManagerDelegate`내에서 `continuation.yield`를 호출할 수 있습니다:

  ```swift
  class AsyncLocationStream: NSObject, CLLocationManagerDelegate {
      lazy var stream: AsyncStream<CLLocation> = {
          AsyncStream { (continuation: AsyncStream<CLLocation>.Continuation) -> Void in
              self.continuation = continuation
          }
      }()
      var continuation: AsyncStream<CLLocation>.Continuation?
  
      func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
  
          for location in locations {
              continuation?.yield(location)
          }
      }
  }
  ```

- 이 접근 방식은 다음 값을 보낼 수 있기 전에 비동기 스트림의 소비자가 값을 완전히 소비할 때까지 기다리지 않습니다. 대신 보내는 모든 값은 기본적으로 비동기 스트림에 버퍼링됩니다.

- 우리는 1개 이상의 위치를 유지하고 싶지 않고 오래된 데이터 처리를 피하기 위해 마지막으로 알려진 위치만 버퍼링하기를 원합니다.

- 비동기 스트림에 버퍼링을 설정하여 이를 수행할 수 있습니다:

  ```swift
  lazy var stream: AsyncStream<CLLocation> = {
      AsyncStream(bufferingPolicy: .bufferingNewest(1)) { (continuation: AsyncStream<CLLocation>.Continuation) -> Void in
          self.continuation = continuation
      }
  }()
  ```

- 사용하는 for루프가 항목을 충분히 빠르게 처리하지 않으면 단일 값만 버퍼링하고 최신 위치만 유지하고 이전 값은 버립니다.

- 스트림이 자연스럽게 종료되면 continuation에서 `finish()`를 호출하여 값 스트림을 종료할 수 있습니다.

- 에러로 인해 스트림이 실패할 수 있는 경우 `AsyncStream`대신 `AsyncThrwoingStream`을 생성하도록 선택할 수 있습니다.

- 스트림에서 에러를 발생시키려면 `finish(throwing:)`을 호출하거나 `Result`객체를 사용합니다.



- `AsyncStream`은 기본적인 것은 복잡하지 않지만 생명 주기를 관리하는 방법에는 신중하게 생각해야 합니다.
- Delegate기반 코드를 연결할 때 continuation이 스트림보다 오래 지속되도록 해서는 안되기 때문에 유의해야 합니다.

<br/>

## Managing your stream's lifecycle

- 기본적으로 비동기 스트림을 종료하는 두 가지 방법이 있습니다. 첫 번째는 더 이상 값을 생성할 수 없기 때문에 자연스럽게 값 생성 종료할 수 있습니다.

- continuation에 `finish`를 호출하고 동시에 수행해야 하는 어떠한 정리들을 제공할 수 있습니다.

- 예를 들어, 실수로 더 이상 사용할 수 없도록 continuation에 nil을 할당할 수 있습니다.

- 혹은 비동기 스트림을 실행하는데 사용되는 작업을 취소하여 스트림을 종료할 수 있습니다:

  <img width="900" alt="CleanShot 2023-03-03 at 16 31 27@2x" src="https://user-images.githubusercontent.com/42647277/222658960-0cbfe2ba-c925-4cdb-9466-b2003ae1bfd3.png">





- 위와 같이 구현하면 새로운 스트림, 새로운 continuation으로 시작하지 않는 한 더 이상 continuation에서 yield를 호출하지 않도록 해야합니다.

- continuation에 `onTermination`을 설정하여 스트림의 마지막을 감지하고 응답할 수 있습니다:

  <img width="899" alt="CleanShot 2023-03-03 at 16 33 59@2x" src="https://user-images.githubusercontent.com/42647277/222659445-4f4584b4-f79c-4c0b-ac92-783f62792364.png">

- 스트림을 취소하거나 스코프를 벗어나는 법 외에도 루프를 벗어나는 방법이 있습니다만 이것은 일반적으로 비동기 스트림을 종료하는 방법이 아니므로 유의하세요.
- 개인적으로 정리하기 가장 쉬운 방법은 async for loop를 중단하는 대신! 스트림 생성 객체에 스트림을 취소하는 메서드를 갖는 것입니다.



- 앞에서 보여준 패턴은 오직 한명의 소비자가 Location스트림 객체를 사용하는 경우에만 작동한다는 점을 명심하는 것도 중요합니다.
- 기본적으로 비동기 시퀀스에는 반복을 여러 루프와 공유할 수 있는 기능이 없기 때문에 스위프트 concurrency에서 단일 스트림을 반복하는 여러 for루프를 가질 수 없습니다.





## 출처

- https://www.donnywals.com/understanding-swift-concurrencys-asyncstream/