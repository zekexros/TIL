# [스위프트] Dynamic Member Lookup
## Dynamic Member Lookup에 대해서 알아봅시다.

제 [미디엄 블로그에서 가져온 글](https://medium.com/@Lionable/스위프트-dynamic-member-lookup-a204912a8fbc) 입니다. 복습할 겸 붙여넣습니다.

Dynamic member lookup은 Swift를 스크립트 언어에 더 가깝게 만들고 심지어 type-safe한 방식으로 수행합니다. 다른언어인 PHP와 Python에서는 이미 사용중인 방식입니다.

<br/>

스위프트라는 언어에 대해 가장 먼저 생각나는 키워드는 ‘안전’입니다. 옵셔널, throw 등의 안전을 위한 문법에 의해 스위프트는 다루기가 더 어려워진 것도 사실입니다.

<br/>

@dynamicMemberLookup이라는 새로운 속성을 통해 스위프트는 subscript 메서드를 호출하여 하위 프로퍼티에 접근하게 됩니다. 정확한 메서드명은   subscript(dynamicMember:)이고 이를 통해 요청한 프로퍼티가 전달되거나 반환할 수 있습니다.

<br/>

예제를 통해 살펴보겠습니다:

```swift
@dynamicMemberLookup
struct Person {
    subscript(dynamicMember member: String) -> String {
        let properties = ["name": "Taylor Swift", "city": "Nashville"]
        return properties[member, default: ""]
    }
}
```

`@dynamicMemberLookup` 속성은 말 그대로 동적으로 멤버를 조회하는 것입니다. 그리고 그 작업을 처리하기 위해 subscript(dynamicMemberLookup:)메서드 구현이 필요합니다. 위 코드를 보시다시피 멤버 이름은 string으로 받아 다시 string으로 반환하는 코드를 작성하였는데 구현부를 살펴보면 dictionary에서 멤버 이름을 조회하고 반환하게 됩니다.

<br/>

그렇다면 이제 Person은 다음과 같이 사용될 수 있습니다:
```swift
let taylor = Person()
print(taylor.name) // Taylor Swift
print(taylor.city) // Nashville
print(taylor.favoriteIceCream) // ""(빈 문자열)
```

신기하죠??!?!?!
`name`, `city`, `favoriteIceCream`이 Person의 프로퍼티로 존재하지 않더라도 에러를 발생시키지 않고 컴파일이 되고 실행이 가능합니다.

그리고 저 멤버(?)들은 컴파일 타임이 아닌, 런타임에 찾게 됩니다.

위의 subscript(dynamicMemberLookup:)메서드는 반드시 string을 반환하도록 구현하였죠, nil을 반환할 가능성이 있는 위험한(?) dictionary를 다룰 때에도 반드시 string을 반환할 수 있도록 `properties[member, default: ""]` 처럼 default로 string이 반환하는 것을 보장해주는 모습입니다. 

동적인 데이터를 다루지만 안정성에 대해 신경써야하는 것은 변함이 없습니다!

subscript(dynamicMemberLookup:)메서드는 다양한 타입을 반환하도록 여러개 선언할 수도 있습니다:

```swift
@dynamicMemberLookup
struct Person {
    subscript(dynamicMember member: String) -> String {
        let properties = ["name": "Taylor Swift", "city": "Nashville"]
        return properties[member, default: ""]
    }
  	subscript(dynamicMember member: String) -> Int {
    	let properties = ["age": 26, "height": 178]
    	return properties[member, default: 0]
    }
}
```

<br/>

### Subscript 문법을 사용하네?

우리가 흔히 사용하는 Array나 Dictionary타입들은 subscript문법에 의해 대괄호를 사용하여 편하게 데이터를 읽거나 쓰고있었습니다. `@dynamicMemberLookup` 또한 subscript메서드를 통해 구현하도록 하는데 정작 사용은 점(.)을 붙여서 사용하도록 구현되어있죠. 이는 개발자들의 편의성을 위해 애플이 dot syntax를 사용하도록 구현한 것이고 실제로는 컴파일러에 의해 대괄호를 사용하는 것으로 해석됩니다.
```swift
// dot syntax을 이용하여 사용하는 모습
someValue.someMember = a
mutateParameter(&someValue.someMember)

// 실제 컴파일러가 해석하는 모습
someValue[dynamicMember: "someMember"] = a
mutateParameter(&someValue[dynamicMember: "someMember"])
```

<br/>

### 어디서 사용되는데? 대체 왜만든거야?
간단하게 점(.)을 붙여 값을 반환받아 사용하는데 컴파일 에러를 발생시키지도 않고 런타임에 찾는 `@dynamicMemberLookup`을 왜만든걸까?

스위프트의 창시자인 Chris는 스위프트와 python간의 상호 운용성을 개선하기 위해 열심히 노력했다고 합니다.

파이썬 뿐만아니라 Ruby, Perl, JavaScript와 같이 동적인 언어와 Swift 사이에 브릿징 코드를 작성할 때, 아니면 다른 proxy-based 코드를 작성할 때 매우 유용할 것 같아 만들었다고 합니다.
(img)(img)(img)(img)

스위프트 5.0에서 도입되는 <@dynamicCallable>은 스크립팅 언어에 대해 더 많은 브릿징을 제공한다고 합니다.

<br/>

### DynamicMemberLookup와 Key paths의 조합
`@dynamicMemberLookup`을 Key paths와 조합하여 사용하면 굉장히 유용합니다.

```swift
@dynamicMemberLookup
class Reference<Value> {
	private(set) var value: Value
  init(value: Value) {
    self.value = value
	}

	subscript<T>(dynamicMember keyPath: KeyPath<Value, T>) -> T {
    value[keyPath: keyPath]
	}
}
```

Reference라는 class타입에 `@dynamicMemberLookup`을 구현하고 Value라는 제네릭 타입도 보입니다. 그리고 subscript(dynamicMember:)에 KeyPath 타입으로 파라미터를 받도록 구현하였네요.

```swift
let reference = Reference(value: Settings())
let theme = reference.colorTheme
```


reference에 점(.)을 찍고 colorTheme을 사용한 모습을 볼 수 있습니다. 즉, KeyPath를 이용하여 Value타입(위에서는 Settings)의 속성이 Reference타입의 속성인 것처럼 직접 접근하여 사용하는 것이 굉장히 유용합니다.

```swift
extension Reference {
    subscript<T>(dynamicMember keyPath: WritableKeyPath<Value, T>) -> T {
        get { value[keyPath: keyPath] }
        set { value[keyPath: keyPath] = newValue }
    }
}

let reference = Reference(value: Settings())
reference.theme = .oldSchool
```

KeyPath타입 대신 WritableKeyPath타입과 getter, setter를 구현하면 값을 직접 변경할 수도 있습니다.

<br/>

### 정리
1. `@dynamicMemberLookup`는 말 그대로 동적으로 멤버를 조회하는 것으로 컴파일 타임이 아닌 런타임에 값을 반환합니다. `@dynamicMemberLookup` 를 타입앞에 선언하고 subscript(dynamicMember:)를 구현하게 되면 dot syntax를 통해 사용하게 됩니다.
2.  KeyPath 혹은 WritableKeyPath타입과 함께 사용하면 좀 더 편리하게 프로퍼티들에 접근하여 데이터를 읽거나 쓸 수 있습니다.
3.  `@dynamicMemberLookup`는 다른 스크립팅 언어들과 함께 사용될 수 있도록 브릿징 코드를 작성하기 위해 태어났습니다.

----
### 참고
<https://www.hackingwithswift.com/articles/55/how-to-use-dynamic-member-lookup-in-swift>
<https://github.com/apple/swift-evolution/blob/master/proposals/0195-dynamic-member-lookup.md>
<https://www.swiftbysundell.com/tips/combining-dynamic-member-lookup-with-key-paths/>
