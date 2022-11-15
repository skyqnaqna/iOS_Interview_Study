# POP에 대해 설명하시오.

# 답변

> POP(Protocol Oriented Programming)는 **참조 타입인 class**에서 가능했던 **상속**을
**protocol**을 사용해 **값 타입**에서도 가능하게 하여,
참조 타입보다 값 타입의 사용을 권장하는 프로토콜 중심의 패러다임입니다.
> 

# 내용

## Protocol

[공식 문서](https://docs.swift.org/swift-book/LanguageGuide/Protocols.html) **|** [공식 문서(한글)](https://bbiguduk.gitbook.io/swift/language-guide-1/protocols)

- 프로토콜은 특정 역할을 하기 위한 메서드나 프로퍼티 등 **요구사항을 정의한 도면**이라고 할 수 있습니다.
- Swift에서는 참조 타입(클래스)과 값 타입(구조체, 열거형) 모두 프로토콜을 채택할 수 있습니다.
- 프로토콜의 요구사항을 충족하는 모든 타입은 프로토콜을 준수(conform)한다고 합니다.
- 프로토콜은 다른 프로토콜을 **상속**할 수 있습니다.
- 클래스는와는 달리 프로토콜은 **다중 상속이 가능**합니다.
- 프로토콜은 **타입**으로 사용할 수 있습니다.
    - **프로토콜 타입의 인스턴스**는 해당 프로토콜을 준수하는 타입의 인스턴스입니다.
    - 함수, 이니셜라이저에서 **매개변수 타입**이나 **반환 타입**으로 사용 가능
    - 프로퍼티, 변수, 상수의 타입으로 사용 가능
    - 배열과 같은 컨테이너 요소의 타입으로 사용 가능
    - Delegation

## 프로토콜 요구사항

> 특정 이름과 타입의 프로퍼티나 메서드를 요구합니다.
> 

### 프로퍼티 요구사항 (Property Requirements)

- 저장 프로퍼티인지 연산 프로퍼티인지에 대한 것은 지정하지 않습니다.
- gettable인지 gettable + settable 인지 지정해야 합니다.
    - gettable + settable : 읽기 전용 저장/연산 프로퍼티 구현 불가능
- 프로퍼티 요구사항은 항상 `var` 키워드를 사용한 변수로 정의합니다.
- 타입 프로퍼티는 `static` 키워드를 사용하고, 클래스에서 `class` 키워드를 사용할 프로퍼티도 `static` 키워드를 사용하여 요구사항을 명시합니다.

```swift
protocol SomeProtocol {
    var mustBeSettable: Int { get set }
    var doesNotNeedToBeSettable: Int { get }
}

protocol AnotherProtocol {
    static var someTypeProperty: Int { get set }
}
```

### 메서드 요구사항 (Method Requirements)

- 메서드의 구현부({ })는 제외합니다.
- 매개변수 기본값을 지정할 수 없습니다.
- 타입 메서드는 `static` 키워드를 사용하고, 클래스에서 `class` 키워드를 사용할 메서드도 `static` 키워드를 사용하여 요구사항을 명시합니다.
- **값 타입**의 [인스턴스 내부의 값을 변경](https://bbiguduk.gitbook.io/swift/language-guide-1/methods#modifying-value-types-from-within-instance-methods)하는 가변 메서드는 `mutating` 키워드를 사용합니다.

```swift
protocol Togglable {
    mutating func toggle()
}
```

### 이니셜라이저(초기화 구문) 요구사항 (Initializer Requirements)

- 이니셜라이저도 메서드처럼 정의만 하고 구현은 하지 않습니다.

```swift
protocol SomeProtocol {
    init(someParameter: Int)
}
```

- 클래스에서 구현할 때는 `required` 수식어를 표시해야 합니다.
    - [필수 초기화 구문(Required Initializers)](https://bbiguduk.gitbook.io/swift/language-guide-1/initialization#required-initializers)

```swift
class SomeClass: SomeProtocol {
    required init(someParameter: Int) {
        // initializer implementation goes here
    }
}
```

- 하지만 상속받을 수 없는 `final` 클래스라면 `required` 수식어를 표시할 필요가 없습니다.
    - 상속할 수 없는 클래스의 이니셜라이저 구현은 무의미하기 때문?
    - [final 수식어에 대한 공식 문서 내용](https://bbiguduk.gitbook.io/swift/language-guide-1/inheritance#preventing-overrides)
- 하위 클래스가 상위 클래스의 지정된 이니셜라이저를 재정의하고 프로토콜의 이니셜라이저를 구현하기 위해서는 `required`와 `override` 식별자를 모두 명시해야 합니다.

```swift
protocol SomeProtocol {
    init()
}

class SomeSuperClass {
    init() {
        // initializer implementation goes here
    }
}

class SomeSubClass: SomeSuperClass, SomeProtocol {
    // "required" from SomeProtocol conformance; "override" from SomeSuperClass
    required override init() {
        // initializer implementation goes here
    }
}
```

### 실패 가능한 이니셜라이저 요구사항 (Failable Initializer Requirements)

- [실패 가능한 이니셜라이저](https://bbiguduk.gitbook.io/swift/language-guide-1/initialization#failable-initializers)에 정의 된대로 실패 가능한 이니셜라이저를 정의할 수 있습니다. `init?`

## 클래스 전용 프로토콜 (Class-Only Protocols)

- 프로토콜의 상속 리스트에 `AnyObject` 키워드를 추가하면, 해당 프로토콜은 클래스 타입만 채택할 수 있습니다.

```swift
protocol SomeClassOnlyProtocol: AnyObject, SomeInheritedProtocol {
    // class-only protocol definition goes here
}
```

- 만약 클래스 타입만 채택 가능한 프로토콜을 구조체나 열거형이 채택하면 컴파일 에러가 발생합니다.

## 확장으로 프로토콜 준수성 추가 (Adding Protocol Conformance with an Extension)

> 프로토콜을 채택하고 준수함으로써 기존 타입을 [확장](https://bbiguduk.gitbook.io/swift/language-guide-1/extensions)할 수 있습니다.
> 

```swift
protocol TextRepresentable {
    var textualDescription: String { get }
}

extension Dice: TextRepresentable {
    var textualDescription: String {
        return "A \(sides)-sided dice"
    }
}

let d12 = Dice(sides: 12, generator: LinearCongruentialGenerator())
print(d12.textualDescription)
// Prints "A 12-sided dice"
```

### 조건적으로 준수 (Conditionally Conforming to a Protocol)

> 제네릭 타입은 특정 조건에서만 프로토콜 요구사항을 충족시킬 수 있습니다.
`where` 절을 사용하여 채택중인 프로토콜 이름 뒤에 제약조건을 작성합니다.
→ [제네릭 Where 절(Generic Where Clauses)](https://bbiguduk.gitbook.io/swift/language-guide-1/generics#where-generic-where-clauses)
> 

```swift
extension Array: TextRepresentable where Element: TextRepresentable {
    var textualDescription: String {
        let itemsAsText = self.map { $0.textualDescription }
        return "[" + itemsAsText.joined(separator: ", ") + "]"
    }
}
let myDice = [d6, d12]
print(myDice.textualDescription)
// Prints "[A 6-sided dice, A 12-sided dice]"
```

- `Array` 의 인스턴스가 `TextRepresentable` 을 준수하는 타입의 요소를 저장할 때마다 `TextRepresentable` 프로토콜을 준수하도록 합니다.

### 확장과 함께 프로토콜 채택 선언 (Declaring Protocol Adoption with an Extension)

> 이미 프로토콜의 요구사항을 준수하지만 채택하지 않은 경우 빈 확장을 사용하여 채택하도록 만들 수 있습니다.
> 

```swift
struct Hamster {
    var name: String
    var textualDescription: String {
        return "A hamster named \(name)"
    }
}
extension Hamster: TextRepresentable {}

let simonTheHamster = Hamster(name: "Simon")
let somethingTextRepresentable: TextRepresentable = simonTheHamster
print(somethingTextRepresentable.textualDescription)
// Prints "A hamster named Simon"
```

## 합성된 구현을 사용하여 프로토콜 채택 (Adopting a Protocol Using a Synthesized Implementation)

> Swift는 `Equatable`, `Hashable`, `Comparable` 에 대해 프로토콜 준수성을 자동으로 제공할 수 있습니다.
합성된 구현을 사용하면 프로토콜 요구사항 구현을 위해 반복적인 코드를 작성할 필요가 없습니다.
> 

다음과 같은 사용자 정의 타입에 대해 `Equatable`, `Hashable` 의 합성된 구현을 제공합니다.

- `Equatable` or `Hashable` 프로토콜을 준수하는 저장된 프로퍼티만 있는 구조체
- `Equatable` or `Hashable` 프로토콜을 준수하는 연관된 타입만 있는 열거형
- 연관된 타입이 없는 열거형

```swift
struct Point: Equatable {
  var x = 0
  var y = 0
}

let point1 = Point(x: 1, y: 1)
let point2 = Point(x: 1, y: 1)

print(point1 == point2) // true
print(point1 != point2) // false
```

- Point 구조체에는 Int 타입 프로퍼티만 존재하고, Int 타입은 `Equatable` 프로토콜을 준수합니다.
- 따라서 `==` 와 `!=` 의 기본 구현을 제공받기 때문에 Point 구조체에 따로 구현할 필요가 없습니다.
- 이를 합성된 구현(Synthesized Implementation)이라고 합니다.

## 프로토콜 구성 (Protocol Composition)

> 프로토콜 구성 `&` 을 사용하면 여러 개의 프로토콜을 준수할 수 있습니다.
여러 개의 프로토콜을 단일 요구사항으로 결합하는 것으로 클래스 타입도 하나만 결합할 수 있습니다.
> 

```swift
protocol Named {
    var name: String { get }
}
protocol Aged {
    var age: Int { get }
}
struct Person: Named, Aged {
    var name: String
    var age: Int
}
func wishHappyBirthday(to celebrator: Named & Aged) {
    print("Happy birthday, \(celebrator.name), you're \(celebrator.age)!")
}
let birthdayPerson = Person(name: "Malcolm", age: 21)
wishHappyBirthday(to: birthdayPerson)
// Prints "Happy birthday, Malcolm, you're 21!"
```

```swift
class Location {
    var latitude: Double
    var longitude: Double
    init(latitude: Double, longitude: Double) {
        self.latitude = latitude
        self.longitude = longitude
    }
}
class City: Location, Named {
    var name: String
    init(name: String, latitude: Double, longitude: Double) {
        self.name = name
        super.init(latitude: latitude, longitude: longitude)
    }
}
func beginConcert(in location: Location & Named) {
    print("Hello, \(location.name)!")
}

let seattle = City(name: "Seattle", latitude: 47.6, longitude: -122.3)
beginConcert(in: seattle)
// Prints "Hello, Seattle!"
```

## 프로토콜 준수 검사 (Checking for Protocol Conformance)

> `is`와 `as` 연산자를 통해 프로토콜을 준수하는지 확인할 수 있습니다.
> 
- `is` 연산자로 해당 인스턴스가 프로토콜을 준수하는지 확입합니다. → true or false
- 다운 캐스팅 연산자 `as?` 로 프로토콜 타입의 옵셔널 값을 반환합니다. → 프로토콜 준수하지 않으면 nil
- 다운 캐스팅 연산자 `as!` 로 프로토콜 타입으로 강제 다운 캐스팅을 합니다. → 실패시 런타임 에러

## 옵셔널 프로토콜 요구사항 (Optional Protocol Requirements)

> 옵셔널 프로토콜은 요구사항을 필수로 구현할 필요가 없는 선택적 요구사항을 정의할 수 있습니다.
> 
- 프로토콜 앞에 `@objc` 속성 추가하기
    - Object-C 클래스를 상속받은 클래스에서만 채택 가능합니다. → 구조체나 열거형은 채택 불가능
- 옵셔널 요구사항 앞에 `@objc` + `optional` 수식어 붙이기
    - 해당 메서드나 프로퍼티는 자동으로 옵셔널이 됩니다.

```swift
@objc protocol CounterDataSource {
    @objc optional func increment(forCount count: Int) -> Int
    @objc optional var fixedIncrement: Int { get }
}

class Counter {
    var count = 0
    var dataSource: CounterDataSource?
    func increment() {
        if let amount = dataSource?.increment?(forCount: count) {
            count += amount
        } else if let amount = dataSource?.fixedIncrement {
            count += amount
        }
    }
}
```

# 추가 질문

## 참조 타입 vs 값 타입

### 참조 타입(Reference Types)

> 힙 영역에 저장되는 타입으로 해당 데이터를 가리키는 주소값은 스택 영역에 저장됩니다.
해당 데이터의 값을 복사하는 것이 아니라 참조값을 전달합니다.
> 
- `class`, `closure`

### 값 타입(Value Types)

> 스택 영역에 저장되는 타입으로 해당 데이터의 값을 복사하여 전달합니다.
> 
- `struct`, `enum`, `tuple`, 기초 타입(Int, Double, String), 컬렉션 타입(Array, Set, Dictionary)

## 클래스에서 static vs class 키워드

### 타입 프로퍼티

> 인스턴스마다 다른 값을 가지는 **인스턴스 프로퍼티**와는 다르게 **타입 자체**에 속하는 프로퍼티를 말한다.
인스턴스 생성 여부와 상관 없으며, 해당 타입의 모든 인스턴스가 공통으로 사용하는 프로퍼티이다.
> 

### 타입 메서드

> **타입 자체에서 호출**이 가능한 메서드.
타입 메서드에서 `self`는 타입 자체를 의미한다.
> 
- **클래스에서** **연산 타입 프로퍼티와 타입 메서드**
    - `static` 으로 정의하면 상속 후 재정의 불가능 → 오버라이딩 X
    - `class` 로 정의하면 상속 후 재정의 가능 → 오버라이딩 O

## OOP?

- 프로그램을 어떻게 설계할 것인가에 대한 방법 중 하나
- 객체 지향 프로그래밍이라고 한다
    - 현실 세계를 프로그램 설계에 반영하는 것을 의미한다
    - 클래스라는 설계도를 이용하여 객체를 생성해낸다
        - 클래스는 속성과 행위를 정의한 것
    - 아이폰을 생산하기 위한 설계도 즉, 클래스가 있다면 이를 공장에서 생산하여 시리얼넘버가 부여된(메모리에 올라간?) 실물 기기를 객체라고 할 수 있다
- 객체지향언어
    - Java, C++, C# …

## OOP의 특징

### 1. 캡슐화

- 데이터와 데이터를 조작하는 함수를 결합하는 것 → 변수와 함수를 묶는 것
- 캡슐화를 통해 정보은닉과 데이터 추상화로 이어진다
- 정보은닉
    - 클래스의 데이터가 다른 클래스에서 숨겨지는 것
    - 클래스에서 접근 지정자를 사용하여 외부에서 볼 수 있는 데이터를 결정한다

### 2. 추상화

- 클래스/구조체의 속성이나 기능을 묶는 것 → 클래스/구조체를 정의하는 것
- 필수적인 정보만 보여주고 세부 사항이나 구현을 숨기는 것
- 헤더 파일의 추상화
    - 헤더 파일에 존재하는 함수의 알고리즘을 몰라도 해당 함수를 호출하여 사용한다

### 3. 상속

- 상위 클래스의 기능을 하위 클래스가 사용할 수 있도록 하여 중복되는 코드의 재사용을 위한 것이다
- 하위 클래스는 자체 속성과 메서드를 추가할 수 있다

### 4. 다형성

- 하나의 작업이 서로 다른 동작을 할 수 있는 것 → 동일한 이름을 가진 작업
- Overloading
    - 메서드 이름은 같지만 매개변수만 다르다
        - 매개변수의 수가 다르거나
        - 매개변수의 타입이 다르거나
        - 매개변수의 순서가 다르거나
- Overriding
    - 상속 관계에 있는 클래스 간에 같은 이름의 메서드를 재정의하는 것
    - 오버라이드 하려는 메서드가 상위 클래스에 존재해야 한다
    - 메서드 이름과 매개변수가 같아야 한다
    - 리턴형도 같아야 한다

## Swift는 OOP인가 POP인가?

> 스위프트는 명령형, 객체지향, 함수형, 프로토콜 지향 프로그래밍의 특징을 갖는 다중 패러다임 언어이다.
> 

## OOP와 POP의 차이?

- 코드의 재사용성을 높인다는 공통점이 있지만 POP는 프로토콜을 이용하여 수평 구조로 타입을 확장하는 방식이고, OOP는 상속을 통해 수직 구조로 타입을 확장하는 방식이다.
- 클래스는 다른 클래스 하나만 상속할 수 있지만 프로토콜을 다중 채택이 가능하다.
- 그래서 클래스를 사용한 추상화는 상속에 의존하기 때문에 슈퍼 클래스가 수정되거나 새로운 중간 클래스를 만들면서 문제가 복잡해질 수 있지만,
프로토콜은 다중 채택을 통해 필요한만큼만 확장과 생성을 할 수 있기 때문에 문제를 분리할 수 있습니다.

# 참고 자료

[https://docs.swift.org/swift-book/LanguageGuide/Protocols.html](https://docs.swift.org/swift-book/LanguageGuide/Protocols.html)

[https://bbiguduk.gitbook.io/swift/language-guide-1/protocols](https://bbiguduk.gitbook.io/swift/language-guide-1/protocols)

[https://www.pluralsight.com/guides/protocol-oriented-programming-in-swift](https://www.pluralsight.com/guides/protocol-oriented-programming-in-swift)

[https://yagom.net/courses/reference-and-value-types-in-swift/lessons/값-타입value-types과-참조-타입reference-types/](https://yagom.net/courses/reference-and-value-types-in-swift/lessons/%ea%b0%92-%ed%83%80%ec%9e%85value-types%ea%b3%bc-%ec%b0%b8%ec%a1%b0-%ed%83%80%ec%9e%85reference-types/)

[https://yagom.net/courses/swift-basic/lessons/프로토콜-지향-프로그래밍-p-o-p/](https://yagom.net/courses/swift-basic/lessons/%ed%94%84%eb%a1%9c%ed%86%a0%ec%bd%9c-%ec%a7%80%ed%96%a5-%ed%94%84%eb%a1%9c%ea%b7%b8%eb%9e%98%eb%b0%8d-p-o-p/)
