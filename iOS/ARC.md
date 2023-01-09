# ARC (Automatic Reference Counting)

# 답변

> 메모리 관리를 위해 참조 타입의 인스턴스에만 적용되는 참조 횟수를 
자동으로 계산해주는 컴파일러 기능입니다.
> 

# 내용

## 동작 원리

인스턴스가 더이상 필요하지 않으면 메모리 할당이 해제되는데, 이때 해당 인스턴스에 접근하려고 하면 앱에 크래시가 발생하거나 데이터가 손상될 것 입니다.
그래서 ARC는 인스턴스가 어떤 프로퍼티, 상수, 변수에서 참조하고 있는지 추적하여 필요한 동안 사라지지 않도록 합니다.

프로퍼티, 상수나 변수에 인스턴스를 할당하면 기본적으로 강한 참조(`strong reference`)를 만드는데,
강한 참조가 남아있으면 메모리 할당 해제를 하지 않기 때문에 **강한 참조**라고 부릅니다.

강한 참조가 발생할 때마다 참조 횟수를 1 증가시키고, 참조가 해제(**nil**)되면 1 감소시키면서 0이 되면 해당 인스턴스는 메모리에서 해제됩니다.

```swift
class Number {
  var name = ""

  init(name: String) {
    self.name = name
    print("Init class: \(self.name)")
  }

  deinit {
    print("Deinit class: \(self.name)")
  }
}

var one: Number? = Number(name: "ONE")
var two: Number? = Number(name: "TWO")
var two2: Number? = two

print("===Step 1===")
one = nil

print("===Step 2===")
two = nil

print("===Step 3===")
two2 = nil
```

```
Init class: ONE
Init class: TWO
===Step 1===
Deinit class: ONE
===Step 2===
===Step 3===
Deinit class: TWO
```

- two와 two2는 이름이 TWO인 클래스 인스턴스를 가리키고 있다. → Reference Counting: 2
- 그래서 two2까지 **nil**을 할당해야 메모리에서 해제된다.

## 강한 참조 사이클 (Strong Reference Cycles, Strong Retain Cycles)

> 두 클래스 인스턴스가 서로 강한 참조를 유지하여 ARC가 각 인스턴스를 해제하지 못하게 되는 것 입니다.
> 

```swift
class A {
  var name = ""
  var classB: B?

  init(name: String) {
    self.name = name
    print("Init class: \(self.name)")
  }

  deinit {
    print("Deinit class: \(self.name)")
  }
}

class B {
  var name = ""
  var classA: A?

  init(name: String) {
    self.name = name
    print("Init class: \(self.name)")
  }

  deinit {
    print("Deinit class: \(self.name)")
  }
}

func foo() {
  var a: A? = A(name: "A")
  var b: B? = B(name: "B")

  a?.classB = b
  b?.classA = a

  a = nil
  b = nil

  print("Return Function")
}

foo()
```

```
Init class: A
Init class: B
Return Function
```

- **Deinit**이 안 된 것을 확인할 수 있다.

|||
|:---:|:---:|
|<img src=https://user-images.githubusercontent.com/31722496/211277588-21b111ff-fe3c-49e7-a2f5-7f57320cd4de.png width=100%> | <img src=https://user-images.githubusercontent.com/31722496/211277554-cd546336-8cd1-4ef1-bb14-b294a470efac.png width=100%>|

- A와 B가 서로를 계속 순환 참조하고 있다.
- 그래서 reference count는 0이 될 수 없고, 메모리에 계속 남아있게 된다. → 메모리 누수 발생

## 강한 참조 해결 방법

> 프로퍼티 속성을 **약한 참조(weak)** 또는 **미소유 참조(unowned)** 로 정의하여 해결할 수 있다.
> 

둘 다 참조 횟수를 증가시키지 않는다.

- **weak**
    - 런타임에 **nil**이 될 수 있다. → 옵셔널 타입
    - 상수에 할당 불가능
- **unowned**
    - 런타임에 **nil**이 될 수 없다.

### 약한 참조 (Weak References)

- 다른 인스턴스의 수명이 더 짧은 경우(= 다른 인스턴스가 먼저 할당 해제) 약한 참조 사용

```swift
class A {
  var name = ""
  var classB: B?

  init(name: String) {
    self.name = name
    print("Init class: \(self.name)")
  }

  deinit {
    print("Deinit class: \(self.name)")
  }
}

class B {
  var name = ""
  weak var classA: A?

  init(name: String) {
    self.name = name
    print("Init class: \(self.name)")
  }

  deinit {
    print("Deinit class: \(self.name)")
  }
}

func foo() {
  let a: A? = A(name: "A")
  let b: B? = B(name: "B")

  a?.classB = b
  b?.classA = a

  print("Return Function")
}

foo()
```

```
Init class: A
Init class: B
Return Function
Deinit class: A
Deinit class: B
```

- 클래스 B의 classA 변수는 약한 참조이기 때문에 순환 참조가 발생하지 않는다.
- 참조하는 인스턴스가 할당 해제되면 ARC는 약한 참조를 **nil**로 변경한다. → 런타임에 변경되는 것
    - 그래서 상수에 할당이 불가능한 것

```swift
func foo() {
  var a: A? = A(name: "A")
  var b: B? = B(name: "B")

  a?.classB = b
  b?.classA = a

  a = nil

  print(b?.classA)

  print("Return Function")
}

foo()
```

```
Init class: A
Init class: B
Deinit class: A
nil
Return Function
Deinit class: B
```

### 미소유 참조 (Unowned References)

- 다른 인스턴스의 수명이 동일하거나 더 긴 경우 사용
- 할당 해제되지 않을 인스턴스를 참조할 경우에 사용
- 할당 해제된 후 미소유 참조 값에 접근하려고 하면 런타임 에러가 발생

```swift
class A {
  var name = ""
  unowned let classB: B

  init(name: String, classB: B) {
    self.name = name
    self.classB = classB
    print("Init class: \(self.name)")
  }

  deinit {
    print("Deinit class: \(self.name)")
  }
}

class B {
  var name = ""
  var classA: A?

  init(name: String) {
    self.name = name
    print("Init class: \(self.name)")
  }

  deinit {
    print("Deinit class: \(self.name)")
  }
}

func foo() {
  let b = B(name: "B")
  let a: A? = A(name: "A", classB: b)

  b.classA = a

  print("Return Function")
}

foo()
```

```
Init class: B
Init class: A
Return Function
Deinit class: B
Deinit class: A
```

- 클래스 A의 변수 classB는 미소유 참조이므로 ARC가 메모리 해제할 수 있다.

```swift
func foo() {
  var b: B? = B(name: "B")

  if let instanceOfB = b {
    instanceOfB.classA = A(name: "A", classB: instanceOfB)
  }

  b = nil

  print("Return Function")
}

foo()
```

```
Init class: B
Init class: A
Deinit class: B
Deinit class: A
Return Function
```

### 미소유 옵셔널 참조 (Unowned Optional References)

- 클래스에 옵셔널 참조를 미소유로 표기할 수 있다. → [Swift 5 Release Notes](https://developer.apple.com/documentation/xcode-release-notes/swift-5-release-notes-for-xcode-10_2?language=objc#:~:text=unowned%20and%20unowned(unsafe)%20variables%20now%20support%20Optional%20types.%20(47326769))
- ARC 소유권 모델 측면에서 미소유 옵셔널 참조와 약한 참조는 같은 컨텍스트에서 사용될 수 있다는데..
미소유 옵셔널 참조는 유효한 객체를 참조하거나 **nil**인지 확인해야 한다고 한다.

```swift
class Department {
    var name: String
    var courses: [Course]
    init(name: String) {
        self.name = name
        self.courses = []
    }
}

class Course {
    var name: String
    unowned var department: Department
    unowned var nextCourse: Course?
    init(name: String, in department: Department) {
        self.name = name
        self.department = department
        self.nextCourse = nil
    }
}

let department = Department(name: "Horticulture")

let intro = Course(name: "Survey of Plants", in: department)
let intermediate = Course(name: "Growing Common Herbs", in: department)
let advanced = Course(name: "Caring for Tropical Plants", in: department)

intro.nextCourse = intermediate
intermediate.nextCourse = advanced
department.courses = [intro, intermediate, advanced]
```
<img src=https://user-images.githubusercontent.com/31722496/211277557-2631b5f7-4bc2-4fa5-9891-a4e1bb127940.png width=80%>

#### ❓ **weak**과 무슨 차이?

```swift
let department = Department(name: "Horticulture")

let intro = Course(name: "Survey of Plants", in: department)
let intermediate = Course(name: "Growing Common Herbs", in: department)
var advanced: Course? = Course(name: "Caring for Tropical Plants", in: department)

intro.nextCourse = intermediate
intermediate.nextCourse = advanced
department.courses = [intro, intermediate, advanced!]

print(intermediate.nextCourse?.name ?? "nextCourse is nil")
// Optional("Caring for Tropical Plants")
```
<img src=https://user-images.githubusercontent.com/31722496/211277562-2e39486f-7223-4217-85f2-35e2a7e0372a.png width=80%>

- 현재 **Department**는 **intro, intermediate, advanced**라는 **Course** 클래스 3개를 갖고 있다. (강한 참조)
- 그리고 **intro**의 nextCourse는 **intermediate**, 
**intermediate**의 nextCourse는 **advanced**로 미소유 옵셔널 참조를 하고 있다.

<img src=https://user-images.githubusercontent.com/31722496/211277539-02c5fbce-68a8-45d4-aef9-8b82f6c55b79.png width=80%>

- 이 상황에서 아래 코드를 추가하면

```swift
department.courses.removeLast()
advanced = nil

print(intermediate.nextCourse?.name ?? "nextCourse is nil")
```

<img src=https://user-images.githubusercontent.com/31722496/211277567-33917010-6bba-43e8-bc08-3132f31c76b0.png width=100%>

- 이렇게 에러가 발생하게 되고, 에러 내용은 다음과 같다.

`Fatal error: Attempted to read an unowned reference but object 0x600000c0c090 was already deallocated2022-11-29 12:42:42.860636+0900
Fatal error: Attempted to read an unowned reference but object 0x600000c0c090 was already deallocated`

- 미소유 참조 객체에 접근하는데 이미 메모리에서 할당 해제되었다는 내용이다.

<img src=https://user-images.githubusercontent.com/31722496/211277569-b4a06284-e962-4825-b859-40ae0aa1e1ec.png width=80%>

<!-- <img src=https://user-images.githubusercontent.com/31722496/211277573-0567a72b-8d4b-4431-9a68-36825f9fe7dc.png width=50% alt="unowned"> -->

![unowned](https://user-images.githubusercontent.com/31722496/211277573-0567a72b-8d4b-4431-9a68-36825f9fe7dc.png)

- 메모리 그래프를 보면 **advanced**가 아직 남아있다. 
왜? 아직 **intermediate**의 nextCourse가 **nil**이 아니고 **advanced**를 가리키고 있기 때문에
- 즉, `nextCourse is nil`이 출력되지 않은 이유는 **intermediate**의 nextCourse는 **nil**이 아니라 **advanced**라서 해당 참조를 타고 메모리에 접근하면 객체가 이미 할당 해제되어 있기 때문에 에러가 발생하는 것이다. → **dangling pointer (해제된 메모리 영역을 가리키고 있는 포인터)**
- `advanced = nil` 로 할당 해제했는데 **intermediate**가 어떻게 가리키고 있는거지??
    - **ARC**가 **unowned** 참조는 **nil**로 바꿔주지 않아서 그렇다.
    런타임에 **nil**이 될 수 없다는 뜻이 바로 이 경우다.

- 이제 nextCourse를 **weak**으로 바꿔보자

```swift
class Course {
    var name: String
    unowned var department: Department
    weak var nextCourse: Course?
    init(name: String, in department: Department) {
        self.name = name
        self.department = department
        self.nextCourse = nil
    }
}

let department = Department(name: "Horticulture")

let intro = Course(name: "Survey of Plants", in: department)
let intermediate = Course(name: "Growing Common Herbs", in: department)
var advanced: Course? = Course(name: "Caring for Tropical Plants", in: department)

intro.nextCourse = intermediate
intermediate.nextCourse = advanced
department.courses = [intro, intermediate, advanced!]

print(intermediate.nextCourse?.name ?? "nextCourse is nil")
// Caring for Tropical Plants

department.courses.removeLast()
advanced = nil

print(intermediate.nextCourse?.name ?? "nextCourse is nil")
// nextCourse is nil
```

![weak](https://user-images.githubusercontent.com/31722496/211277575-6d7b069b-0a35-4bc2-ab94-c3c8f3c23b45.png)

- 에러 없이 잘 출력되고, 메모리에도 남아있지 않은 것을 볼 수 있다.
- 메모리 그래프에서 다른 점은 `SwiftWeakRefStorage`를 가리키고 있다는 건데..
Course의 인스턴스가 참조하고 있는 것이 무엇인지, 그리고 참조하는 객체의 참조 횟수 정보를 갖고 있다.
- 이것이 ARC가 수행하는 추적, 관리 기능인 것으로 추측된다.

<img src=https://user-images.githubusercontent.com/31722496/211277576-41beeb44-1456-497e-b016-a6d050672e7a.png width=80%>

<img src=https://user-images.githubusercontent.com/31722496/211277582-99100e28-7510-40ef-8f29-a80a3047e376.png width=80%>

- **weak**으로 설정하면 **advanced**가 메모리에서 할당 해제될 때 **intermediate**의 nextCourse는 **nil**로 바뀌기 때문에 에러가 발생하지 않는다!

#### ❓ 그럼 weak을 쓰면 되는 것 아닌가?? 왜 미소유 참조에 옵셔널 타입을 지원한거지??

- 위 실험으로 알 수 있는 것은 ARC가 **nil**로 바꿔주냐, 안 바꿔주냐 차이가 있다는 것이다.
- **nil**로 바꾸기 위해서는 ARC가 계속 추적하고 관리해야한다. 그러면 할 일이 더 생기는 것이고 **오버헤드**로 이어질 수 있다.
- 따라서 참조할 인스턴스가 아예 없을수도 있지만(nil) 만약 추가된다면 나보다 먼저 사라지지 않을 경우에 **unowned**를 사용하고, 사라질 수도 있으면 **weak**을 사용하는 것이 바람직해 보인다.

### 미소유 참조와 암시적 추출 옵셔널 프로퍼티 (Unowned References and Implicitly Unwrapped Optional Properties)

**서로 강한 참조하고 있는 두 인스턴스의 강한 참조 사이클 해결하기**

1. 둘 다 nil이 될 수 있는 경우 → [약한 참조로 해결](https://www.notion.so/ARC-Automatic-Reference-Counting-96258d7ce5fd4cce89b669793e963572)
2. 하나는 nil이 될 수 있고 다른 하나는 nil이 될 수 없는 경우 → [미소유 참조로 해결](https://www.notion.so/ARC-Automatic-Reference-Counting-96258d7ce5fd4cce89b669793e963572)
    - 다른 하나는 처음에 nil일 수 있지만 값을 가지게 되면 nil이 될 수 없는 경우 → [미소유 옵셔널 참조로 해결](https://www.notion.so/ARC-Automatic-Reference-Counting-96258d7ce5fd4cce89b669793e963572)
3. 둘 다 초기화가 완료되면 nil이 되어서는 안되는 경우 → **미소유 참조와 암시적 추출 옵셔널 프로퍼티**
    - 이 경우 한 클래스의 미소유 프로퍼티를 다른 클래스의 암시적으로 추출된 옵셔널 프로퍼티와 결합하는 것이 유용하다.
    - 초기화가 완료되면 옵셔널 추출 없이 서로 직접 접근 가능하고 참조 사이클도 피할 수 있기 때문이다.

```swift
class Country {
    let name: String
    var capitalCity: City!
    init(name: String, capitalName: String) {
        self.name = name
        self.capitalCity = City(name: capitalName, country: self)
    }
}

class City {
    let name: String
    unowned let country: Country
    init(name: String, country: Country) {
        self.name = name
        self.country = country
    }
}
```

- City의 초기화 구문은 Country 인스턴스를 갖고 있는데 Country 초기화 구문에서 City의 초기화 구문이 호출된다. 그래서 Country의 인스턴스가 초기화 될 때까지 City초기화 구문에 `self`를 전달할 수 없다. ([2단계 초기화](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html#ID220))
- 이 문제를 해결해기 위해 Country의 capitalCity 프로퍼티를 암시적 추출 옵셔널 프로퍼티로 선언한다. → `City!`
- 그럼 capitalCity는 **nil**을 기본 값으로 가지고 있고, Country 인스턴스는 name 프로퍼티를 설정하는 순간 초기화 된 것으로 간주된다. 따라서 name 프로퍼티가 설정되는 순간 암시적 self 프로퍼티를 전달할 수 있게 된다.
- 그 결과 강한 참조 사이클 없이 Country와 City 인스턴스를 생성하고 capitalCity 프로퍼티는 옵셔널 값을 추출할 필요없이 직접 접근할 수 있다.

```swift
var country = Country(name: "Canada", capitalName: "Ottawa")
print("\(country.name)'s capital city is called \(country.capitalCity.name)")
// Prints "Canada's capital city is called Ottawa"
```

## 클로저에 대한 강한 참조 사이클

> 강한 참조 사이클은 클래스 인스턴스의 프로퍼티에 클로저를 할당하고 해당 클로저 바디에서 인스턴스를 붙잡는(캡쳐) 경우에도 발생할 수 있다.
> 

클로저의 캡쳐는 바디 안에서 `self.someProperty`와 같이 인스턴스 프로퍼티에 접근하거나 메서드를 호출하면서 외부 **context**를 참조하는 것 입니다.

클로저는 클래스처럼 참조 타입이다. 그래서 프로퍼티에 클로저를 할당하면 해당 클로저를 참조하게 되는 것이다.

```swift
class HTMLElement {

    let name: String
    let text: String?

    lazy var asHTML: () -> String = {
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }

    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }

    deinit {
        print("\(name) is being deinitialized")
    }

}

var paragraph: HTMLElement? = HTMLElement(name: "p", text: "hello, world")
print(paragraph!.asHTML())
// Prints "<p>hello, world</p>"
```

- asHTML은 String 값을 반환하는 클로저 타입이다.

<img src=https://user-images.githubusercontent.com/31722496/211277584-a8e54448-cfd5-403a-bcf7-80e8c4886002.png witdh=80%>

- HTMLElement 인스턴스와 asHTML 값으로 사용된 클로저 간에 강한 참조 사이클이 생성된다.
    - 인스턴스의 asHTML 프로퍼티는 클로저에 대해 강한 참조를 하고 있다.
    - 클로저 내의 self.text와 self.name에서 `self`를 참조하기 때문에 클로저는 HTMLElement ****인스턴스를 강한 참조한다. → [캡처](https://docs.swift.org/swift-book/LanguageGuide/Closures.html#ID103)
        - 클로저는 클로저가 정의된 주변 컨텍스트에서 상수와 변수를 캡처할 수 있다. 캡처한 상수와 변수를 정의한 컨텍스트가 사라져도 해당 상수와 변수 값을 참조하고 수정할 수 있다.

### 클로저에 대한 강한 참조 사이클 해결

- 클로저의 **캡처 목록**을 정의하여 클로저와 인스턴스 간의 강한 참조 사이클을 해결한다.
- 캡처 목록은 클로저 내에서 참조 타입을 캡처할 때 사용할 규칙을 정의한다.
    - 각 참조를 약한 참조하거나 미소유 참조로 선언

```swift
lazy var someClosure = {
    [unowned self, weak delegate = self.delegate]
    (index: Int, stringToProcess: String) -> String in
    // closure body goes here
}
```

- 클로저와 캡처한 인스턴스가 항상 서로를 참조하고 수명이 같으면 클로저의 캡처를 미소유 참조로 정의한다.
- 반면에 캡처된 참조가 **nil**이 될 수 있을 때 약한 참조로 정의한다.

```swift
class HTMLElement {

    let name: String
    let text: String?

    lazy var asHTML: () -> String = {
        [unowned self] in
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }

    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }

    deinit {
        print("\(name) is being deinitialized")
    }

}

var paragraph: HTMLElement? = HTMLElement(name: "p", text: "hello, world")
print(paragraph!.asHTML())
// Prints "<p>hello, world</p>"

paragraph = nil
```

```
<p>hello, world</p>
p is being deinitialized
```

- 미소유 참조로 `self`를 캡처하여 강한 참조 사이클을 해결했다.
- 그래서 paragraph에 **nil**을 설정하면 HTMLElement 인스턴스가 할당 해제된다.

<img src=https://user-images.githubusercontent.com/31722496/211277586-720f50d8-8e9e-440d-8522-6814bf7902f9.png width=80%>

# 추가 질문

## ARC와 Garbage Collection 차이?

- 참조 카운팅 시점
    - ARC는 컴파일 시, GC는 프로그램 동작 중
- ARC는 컴파일 시 해제 시점이 정해지므로 언제 해제될지 예측 가능 → 메모리 관리를 위한 추가 자원이 필요 없음
- 그래서 ARC는 작동 규칙을 알아야 메모리 누수를 방지할 수 있음
- GC는 복잡한 상황에서도 인스턴스를 해제할 수 있음 → 메모리 감시를 위한 추가 자원이 필요
- GC는 규칙에 신경 쓸 필요가 없음 → 언제 해제될지 예측하기 어려움

## ARC는 컴파일 시 실행되는데 어떻게 동적으로 실행되는 것들의 참조 횟수를 관리할 수 있는가?

[https://sujinnaljin.medium.com/ios-arc-뿌시기-9b3e5dc23814](https://sujinnaljin.medium.com/ios-arc-%EB%BF%8C%EC%8B%9C%EA%B8%B0-9b3e5dc23814)

# 참고 자료

[https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)

[https://developer.apple.com/library/archive/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html](https://developer.apple.com/library/archive/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html)

[https://sujinnaljin.medium.com/ios-arc-뿌시기-9b3e5dc23814](https://sujinnaljin.medium.com/ios-arc-%EB%BF%8C%EC%8B%9C%EA%B8%B0-9b3e5dc23814)

[https://zeddios.tistory.com/1214](https://zeddios.tistory.com/1214)
