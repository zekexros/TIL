# Making async system APIs backward compatible

- 스위프트 5.5의 새로운 동시성 시스템은 이전 버전과 호환되지만, 새로운 동시성 기능을 사용하는 기본 API중 일부는 여전히 iOS15, macOS Monterey 및 나머지 애플의 2021년 운영 체제에서만 사용할 수 있습니다.

- 예를 들어 `URLSession`의 data task API의 새로운 async/await 버전을 사용하려고 하면 컴파일러 에러가 발생합니다:

  <img width="808" alt="CleanShot 2023-03-12 at 23 32 23@2x" src="https://user-images.githubusercontent.com/42647277/224551404-b3d4902f-f694-4ba9-b1d6-3f709f40690d.png">



- 다행인 것은 스위프트의 새로운 동시성 시스템이 기존 코드를 async/await로 바꿀수 있도록 continuation 메커니즘을 함께 제공하기 때문에 우리가 코드를 직접 고쳐 사용할 수 있다는 것입니다.

<br/>

- 위의 async/await로 되어있는 `URLSession`API를 iOS 13까지 사용할 수 있도록 하는 방법은 아래와 같습니다:

  <img width="808" alt="CleanShot 2023-03-12 at 23 37 24@2x" src="https://user-images.githubusercontent.com/42647277/224551647-64e8feb2-2bf5-435a-8046-cd1910abc028.png">

  > 앱의 배포 타겟을 iOS15이상일 때 컴파일 에러를 받을 수 있도록 위의 extension에 커스텀으로 deprecation주석을 추가하는 것에 유의하세요.

- 위 코드로 인해 옛날 버전의 코드가 주를 이루는 곳에서도 async/await를 사용하여 `URLSession`기반 네트워크 호출을 쉽게 수행할 수 있고, `ModelLoader`내의 컴파일러 에러가 사라졌습니다.

- 그러나 위의 예제 코드는 cancellation을 다루지 않습니다. 보다 완전하게 이전 버전과 호환되는 `URLSession`API를 알고싶다면 [AsyncCompatibilityKit over on GitHub](https://github.com/JohnSundell/AsyncCompatibilityKit/blob/main/Sources/URLSession%2BAsync.swift)를 확인하세요.

<br/>

- 위와 같은 작업이 `async`로 표시된 모든 API에 필요한 작업은 아닙니다.
- 사실 completion handler기반의 Objective-C API들로부터 자동으로 번역된 것들은 자동으로 이전 버전과 호환되도록 만들어졌습니다.(마치 `Task`와 `AsyncStream`처럼 표준 라이브러리에 속하는 타입뿐만 아니라 동시성 기능 자체과 같이).



## 출처

- https://www.swiftbysundell.com/articles/making-async-system-apis-backward-compatible/