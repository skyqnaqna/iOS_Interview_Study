# 답변

> 변수나 상수 등에 값이 있을수도 없을수도 있다는 것을 표현한 타입이며, 보통 변수 안에 값이 명확히 있는지 확신할 수 없을 때 주로 사용합니다. 옵셔널 변수에 아무값도 할당하지 않으면 nil이 할당됩니다.
> 
- 공식문서 : 옵셔널이란 wrapped value 또는 nil(값이 없는)를 표현하기 위한 타입
- 옵셔널 타입의 매개변수만을 보고 이 값은 nil일 수도 있는 값이구나를 API문서를 보지 않고 알 수 있다.

# 내용

## 옵셔널은 어떻게 구현되어 있을까?

> Optional은  none과 some 케이스를 가진 `enum` 타입이며 `Generic`으로 구현되어 있습니다. `ExpressibleByNilLiteral` 프로토콜도 따른다.


우리는 옵셔널은 ?로 축약해서 사용하지만, 정식으로 옵셔널 타입임을 표현하려면 이렇게 표현한다.
아하! 그래서 옵셔널의 정식표현에서 타입을 넣을 수 있었고(Generic), nil이나 값 둘 중 하나를 할당할 수 있었구나(enum)


```swift
let name: String? = nil //축약표현
let name: Optional<String> = nil //정식표현
```


- 이때 none은 값이 없고, some(Wrapped)는 값이 있음을 나타내는 상태이다.

```swift
@frozen enum Optional<Wrapped>: ExpressibleByNilLiteral {
  case none
  case some(Wrapped)
}
```

<details>
<summary>switch문을 통해 optional 값을 확인할 수 있다.</summary>
<div markdown="1">
    
    ```swift
    func checkOptionalValue(optionalValue: Any?) {
        switch optionalValue {
        case .none:
            print("Optional value is nil")
        case .some(let value):
            print("Value is \(value)")
        }
        
    }
    
    var name: String? = "Jiin"
    checkOptionalValue(optionalValue: name) //Value is Jiin
    
    var none: String? = nil
    checkOptionalValue(optionalValue: none) //Optional value is nil
    ```
    
</details>
  
### enum이란?

> 관련된 값의 그룹에 대한 공통 타입을 정의하며, 코드 내에서 type-safe한 방식으로 값을 동작하게 한다.
> 

* type-safe : enum을 사용함으로써 원하지 않는 값이 잘못 입력되거나 , 이미 정의되어 제한된 값 중 선택을 하도록하여 **잘못된 자료가 입력되는 것을 방지**

- CaseIterable Protocol을 채택하면 enum을 하나의 배열처럼 사용할 수 있다.

```swift
enum CompassPoint : CaseIterable {
    case north		
    case south		
    case east		
    case west
}

print(CompassPoint.allCases.count) // 4
for c in CompassPoint.allCases {
	print(c)
}
// north
// south
// east
// west
```

### Generic이란?

> 타입에 의존하지 않는 범용 코드를 작성할 때 사용하며, 제네릭을 사용하면 중복을 피하고 코드를 유연하게 사용할 수 있다.
> 

우리가 흔히 쓰는 Array나 Dictionary 또한 제네릭 컬렉션이다.
( *CollectionType: Array, Set, Dictionary )

Array에 Int형 또는 String형 배열을 만들 수 있었던 것도 제네릭이기 때문에 타입에 제한 없이 사용할 수 있었던 것 !

### IUO(Implicitly Unwrapped Optional)이란?

= 옵셔널 묵시적(암시적) 추출 

> 별도의 추출 과정을 거치지 않고 자동으로 옵셔널이 해제되는 것이다.
> 

암시적 추출 옵셔널도 옵셔널이므로 nil이나 값을 가질 수 있다.  주로 nil로 시작하지만, 값을 가질 거란 확신(~~위험한 생각~~)이 있을 때 주로 사용한다.

옵셔널 타입은 뒤에 ?를 사용하지만 `let name: String?`
암시적 추출 옵셔널은 타입 뒤에 !를 붙여 사용한다. `let name: String!`

이미 추출된 것처럼 동작하기 때문에 if-let 구문을 사용할 필요가 없고, 일반 값처럼 사용할 수 있지만 
nil이 할당되어 있을 때 접근 시도하면 런타임 오류 발생한다.

```swift
var strNonOptional: String = ""
var strImplicitlyUnwrappedOptional: String! = nil

strNonOptional += "abc" //<- Works without problem

strImplicitlyUnwrappedOptional += "abc" //<- Cause your app crash
```

IUO도 옵셔널이기 때문에 출력하면 옵셔널로 감싸서 출력된다.

- Optional Type을 Non-Optional Type에 대입할 때 별도의 추출 과정 없이 대입 가능하다.

```swift
var iuoHello: String! = "Hello"
print(iuoHello) //Optional("Hello")

//옵셔널 타입이 아닌 변수에 IUO 타입의 값을 할당하면 옵셔널이 암묵적 추출된다.
var temp: String = iuoHello 
print(temp) //Hello 
```
<img width="723" alt="스크린샷 2022-11-01 오후 2 46 35" src="https://user-images.githubusercontent.com/37897873/199659991-a2a7538c-bf0c-4b90-9946-5b4e6e076e6f.png">


### 그럼 언제 IUO를 사용할까?

1. @IBOutlet에서도 IUO 사용한다 : **프로퍼티 지연 초기화** 를 위해 사용
   ![Untitled](https://user-images.githubusercontent.com/37897873/199660135-8841d242-94ae-436c-a8ff-abb0fd9f1e9d.png)

2. API에서 IUO를 리턴한 경우 (라는데 이건 잘 모르겠다)

# 추가 질문

## Optional Unwrapping 하기 위한 방식은 무엇인가요?

> some 케이스로 숨어져 있는 옵셔널 값 → 옵셔널 아닌 값을으로 추출하는 것
> 

### 1) 강제 추출

- 옵셔널을 강제로 추출하여 반환해줍니다. 간단하지만 가장 위험한 방식이라 꼭 필요한 상황 아니면 사용을 지양합니다. (런타임에서 오류가 일어날 가능성이 높기 때문에)
- 항상 옵셔널 값이 nil이라는 확신이 있을 때만 `!` 를 붙여 사용
- 강제추출시 옵셔널에 값이 없다면, nil이라면 런타임 오류가 발생한다.
<img width="510" alt="스크린샷 2022-11-01 오후 1 41 25" src="https://user-images.githubusercontent.com/37897873/199660252-10cb0504-464b-4fb2-aa63-dfaf7d1e0e12.png">

### 2) 옵셔널 바인딩

if, guard구문을 통해 값이 nil인지 먼저 확인한다. 좀 더 안전한 방식
nil인 경우, 값이 있는지 경우에 따라 결과를 다르게 하고싶을 때 사용

```swift
var myName: String? = "jiin"
var jiin = myName!

//if let
if let name = myName {
    print("My name is \(name)")
} else {
    print("name is nil!!")
}
// 결과 : My name is jiin

//if var
if var name = myName {
    name = "julia" //변수니까 값 변경 가능
    print("My name is \(name)")
} else {
    print("name is nil!!")
}
//결과 : My name is julia
```

### 3) 옵셔널 체이닝

wrapped instance의 속성이나 메소드에 safely 하게 접근하기 위해 `?` 연산자를 후위에 두어 사용한다.

- 옵셔널 타입의 객체의 속성값에 접근할 때 `?` 를 붙이지 않으면, 이런 에러문구가 발생한다.
- "옵셔널 타입의 값에 접근하려면 반드시 unwrapped 되어야 합니다. `?` 로 옵셔널 체이닝 방식을 쓰거나 `!`로 강제 언래핑 해라"

<img width="645" alt="스크린샷 2022-11-01 오후 2 16 38" src="https://user-images.githubusercontent.com/37897873/199660317-51c7f265-1ed8-403f-9256-a27b3b5796af.png">

### 4) Nil-Coalescing 연산자 사용

Optional instance가 nil인 경우 `??` 를 사용해서 기본 값을 제공한다.

```swift
var name: String? = "jiin"
var nilName: String? = nil

print(name ?? "지인") //이름이 있으므로 jiin이 추출되고
print(nilName ?? "익명") //이름이 nil이므로 기본 값인 익명이 출력됐다.
```

# 참고 자료

### 공식문서

- [https://developer.apple.com/documentation/swift/optional](https://developer.apple.com/documentation/swift/optional)
- [https://docs.swift.org/swift-book/LanguageGuide/Enumerations.html](https://docs.swift.org/swift-book/LanguageGuide/Enumerations.html)
- [https://docs.swift.org/swift-book/LanguageGuide/Generics.html](https://docs.swift.org/swift-book/LanguageGuide/Generics.html)

### 구글링

- Optional : [https://babbab2.tistory.com/132](https://babbab2.tistory.com/132)
- enum : [https://velog.io/@parkgyurim/Swift-enumeration](https://velog.io/@parkgyurim/Swift-enumeration)
- 컴파일 오류 vs 런타임 오류 : [https://velog.io/@un1945/typeOfError](https://velog.io/@un1945/typeOfError)
- 옵셔널과 타입캐스팅 : [https://ujeon.medium.com/swift-옵셔널과-타입캐스팅-c41944937d43](https://ujeon.medium.com/swift-%EC%98%B5%EC%85%94%EB%84%90%EA%B3%BC-%ED%83%80%EC%9E%85%EC%BA%90%EC%8A%A4%ED%8C%85-c41944937d43)