# Escaping Closures

- function의 인자로써 클로저가 사용될 때 @escaping 키워드가 사용되고, 이 키워드는 클로저의 life cycle을 나타냅니다.
- @escaping 키워드가 붙은 클로저는 함수의 스코프를 탈출하여 실행될 수 있다는 뜻입니다.
- @escaping 키워드가 붙지 않으면 일반적으로는 non-escaping한 클로저 입니다. 이 클로저는 함수의 스코프가 끝남과 동시에 lifet cycle이 종료됩니다.


<br/>

## 클로저가 탈출하는 방법
### 변수에 저장하기
- 클로저가 탈출하는 방법을 구현하는 것은 간단합니다. function이 끝난 후에도 클로저가 살아남게 하려면 함수 바깥에 있는 변수에 저장하면 됩니다.
- 아래는 탈출 클로저의 이해를 돕기위한 예제 코드입니다. 다소 길어보일 수 있지만 천천히 살펴보면 충분히 이해할 수 있습니다.
```swift
import Foundation
import CoreLocation

class MyLocationManager: NSObject, CLLocationManagerDelegate {
    let locationManager: CLLocationManager
    
    private var completionHandler: ((_ location: CLLocation) -> Void)? // <1>
    
    override init() {
        locationManager = CLLocationManager()
        super.init()
        locationManager.delegate = self
    }
    
    // <2>
    func getCurrentLocation(_ completion: @escaping (_ location: CLLocation) -> Void) { 
        completionHandler = completion
        locationManager.requestLocation()
    }
    
    // MARK: - CLLocationManagerDelegate
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        if let location = locations.first {
            completionHandler?(location) // <4>
            completionHandler = nil // <5>
        }
    }
    
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {}
}
```
1. CLLocationManager와 CLLocationManagerDelegate를 통해 현재 좌표를 얻고 좌표를 바탕으로 completionHandler를 실행하는 것이 목표입니다.
2. <2>번을 보면 locationManager.requestLocation()을 통해 위치를 얻어옵니다. 이 때, completion handler는 getCurrentLocation() 함수의 생명주기와 같이 끝나선 안됩니다. Delegate함수인  locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation])함수를 통해 위치를 가져올 때까지 살아있어야 하죠. 그리고 그 위치를 클로저에 담아서 실행시켜야 합니다.
3. 목적을 위해 <1>의 completionHandler: ((_ location: CLLocation) -> Void)?에 저장하여 클로저를 탈출시켜 오래 살 수 있도록 합니다.
4. <4>를 보면 살려놓은 탈출 클로저가 담긴 completionHandler가 드디어 실행됩니다. 그리고 목적을 이뤘으니 <5>처럼 nil을 할당하여 메모리 관리까지 신경써준 모습입니다.

### Nested escape
- 클로저를 탈출시키는 다른 방법은 바로 탈출 클로저 안에 클로저를 사용하는 것입니다.


```swift
func delay(_ closure: @escaping () -> Void) { // <1>
    DispatchQueue.main.asyncAfter(wallDeadline: .now() + 3) {
        closure() // <2>
    }
}
```
  - DispatchQueue의 asyncAfter가 탈출 클로저이기 때문에 closure인자는 @escaing으로 선언되어야 합니다.

  <br>

## 클로저의 탈출 여부가 중요한 이유
- 위의 클로저가 탈출하는 방법을 통해 알 수 있듯, 탈출 클로저는 어딘가에 저장되어있고 이는 강한 순환 참조를 발생시킬 수 있기에 조심해야 합니다.
- 그렇기에 @escaping 키워드는 ⚠️조심하라는 경고와 같습니다.
- 탈출 클로저에 의한 강한 순환 참조가 발생하는 예제 코드는 다음과 같습니다.
```swift
class DetailViewController: UIViewController {
    let locationManager = MyLocationManager() // <1>
    
    override func viewDidLoad() {
        super.viewDidLoad()

        locationManager.getCurrentLocation { (location) in
            print("Get location: \(location)")
            self.title = location.description // <2>
        }
    }
}
```
1. DetailViewController는 <1>을 통해 locationManager를 가지고 있습니다.
2. locationManager의 getCurrentLocation 클로저 내부를 보면 self가 보이죠. 네, self(DetailViewController)를 참조하였습니다. getCurrentLocation는 MyLocationManager가 가지고있는 클로저 입니다.
3. DetailViewController이 얌전히 locationManager를 들고있었으면 아무 문제가 없었을텐데... 탈출 클로저를 쓰는 바람에 locationManager가 DetailViewController를 참조하는 그림이 그려지고 말았습니다.

## non-escaping을 기본으로 설정한 이유
- 여러가지 장점이 있는데 그중 가장 큰 장점은 <U>퍼포먼스</U> 와 <U>컴파일러</U>에 의한 코드 최적화 입니다.
- non-escaping 클로저는 함수가 반환될 때 캡처한 모든 객체들을 해제할 것이 분명하기 때문에(함수 바깥으로 나갈일이 없기에 함수가 끝날 때 쯤이면 클로저는 모두 실행되고 캡쳐된 객체들은 해제되어 있음) 컴파일러는 retain/release를 계산하기 쉽습니다.
- 귀찮게 weak self를 사용할 필요가 없습니다.

<br/>

## 결론
- @escaping 키워드는 함수의 매개변수로 이용되는 클로저가 함수보다 더 오래 살고싶을 때 사용됩니다.
- @escaping이 안붙으면 기본적으로 non-escaping클로저 입니다. 함수의 스코프 내에서 실행되거나 안되거나 맘대로 하지만 절대 함수 바깥에서 살아있을 수 없습니다.
- 강한 순환 참조를 발생시킬 수 있으므로 주의해야 합니다.

<br/>

----
출처
- https://docs.swift.org/swift-book/LanguageGuide/Closures.html#ID546
- https://sarunw.com/posts/what-is-escaping-in-swift-closures/
- https://medium.com/swiftcommmunity/what-do-mean-escaping-and-nonescaping-closures-in-swift-d404d721f39d

