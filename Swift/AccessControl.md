# 접근제어자가 무엇이며 왜 필요한가요?

# 답변

> 접근 제어는 다른 소스 파일이나 모듈에서 코드에 접근하는 것을 제한하는 것으로,
상세 구현을 숨기고 해당 코드에 접근하고 사용할 수 있는 인터페이스를 지정할 수 있습니다.
이는 정보 은닉을 하여 불필요한 접근을 막고(부작용 방지) 시스템을 잘 분리하기 위한 것입니다.
> 

# 내용

## 모듈

- 단일 단위로 빌드되고 제공되는 프레임워크
- 애플리케이션과 같은 코드 배포의 단일 단위
- `import` 키워드 사용

## 소스 파일

- 모듈 내 단일 소스 코드 파일

## 접근 수준

### Public

- 구현된 소스 파일뿐만 아니라 외부 모듈까지 접근 가능합니다.
- 주로 프레임워크에서 외부와 연결될 인터페이스 구현에 쓰입니다.

### Open

- Public과 같은 접근 수준이지만 클래스와 클래스 멤버에서만 사용 가능합니다.
- 다른 접근 수준의 클래스/클래스 멤버는
    - 그 클래스/클래스 멤버가 정의된 모듈 내에서만 상속/재정의 가능 ↔ Open은 외부 모듈에서도 상속/재정의 가능
- 해당 클래스를 다른 모듈에서도 부모클래스로 사용하겠다는 뜻

### Internal (Default)

- 기본 접근 수준으로 정의된 모듈 내에서만 접근 가능합니다.
- 외부 모듈에게는 숨기고 내부에서는 전역으로 사용할 경우에 지정합니다.

### File Private

- 구현된 소스 파일 내부에서만 사용 가능합니다.
- 다른 소스 파일에서 접근하면 부작용이 생길 수 있는 경우에 사용합니다.

### Private

- 정의하고 구현한 범위 내에서만 사용 가능합니다.
- 같은 소스 파일 안에 있더라도 다른 타입이나 기능에서는 사용할 수 없습니다.

⭐ (**접근 수준 높음)** **public, open > internal > fileprivate > private** (**접근 수준 낮음)**

<img src=https://user-images.githubusercontent.com/31722496/211302632-11308e86-0a4d-48cb-ba9e-cc837ee90f4f.png width=80%>

## 기본 원칙

> 상위 요소보다 접근 수준이 높은 하위 요소를 정의할 수 없다.
함수는 파라미터 타입과 반환 타입보다 높은 접근 수준을 가질 수 없다.
상수, 변수, 프로퍼티는 타입보다 높은 접근 수준을 가질 수 없다.
> 

### ex)

private 클래스의 멤버 변수/메서드가 public → X

public 튜플, 튜플 내부 요소는 private → X (튜플의 내부 요소 접근 수준보다 높은 접근 수준을 가질 수 없다.)

private 타입을 public 프로퍼티로 작성하는 것 → X

열거형의 케이스는 열거형과 같은 접근 수준을 가진다. → 케이스마다 다른 접근 수준 지정 불가능

## 서브클래싱

> 하위 클래스는 상위 클래스의 접근 수준보다 높은 접근 수준을 가질 수 없다.
하지만 상속된 클래스 멤버를 상위 클래스 버전보다 높은 접근 수준을 가질 수 있게 재정의 할 수 있다.
> 

```swift
public class A {
    fileprivate func someMethod() {}
}

internal class B: A {
    override internal func someMethod() {}
}
```

- A를 상속한 B는 접근 수준이 낮아졌다. (public > internal)
- 하지만 override한 someMethod()는 접근 수준이 높아졌다. (fileprivate < internal)

- 상위 클래스의 멤버에 대한 호출이 허용된 접근 수준 컨텍스트 내에서, 하위 클래스 멤버보다 낮은 접근 권한을 가진 상위 클래스 멤버 호출이 가능하다.
    - 같은 소스 파일 안에서 상위 클래스의 fileprivate 멤버 호출 or 같은 모듈 안에서 상위 클래스의 internal 멤버 호출하는 경우

```swift
public class A {
    fileprivate func someMethod() {}
}

internal class B: A {
    override internal func someMethod() {
        super.someMethod()
    }
}
```

- 상위 클래스 A와 하위 클래스 B는 같은 소스 파일에 정의됐으므로 super.someMethod()를 호출하기 위한 someMethod()의 구현은 유효하다.

## Getter와 Setter

> 상수, 변수, 프로퍼티, 서브스크립트에 대한 getter와 setter는 자동으로 같은 접근 수준을 갖습니다.
하지만 읽기-쓰기 범위를 제한하기 위해 즉, 읽기 전용으로 제한하기 위해 setter만 더 낮은 접근 수준을 갖도록 할 수 있습니다.
> 

```swift
public struct TrackedString {
    public private(set) var numberOfEdits = 0
    public var value: String = "" {
        didSet {
            numberOfEdits += 1
        }
    }
    public init() {}
}
```

- numberOfEdits의 getter는 public이고 setter는 private으로 명시하여 구조체 정의 외부에서는 읽기 전용 프로퍼티로 사용할 수 있습니다.

# 추가 질문

## 정보 은닉에 대해

정보 은닉이 필요한 이유? 정보 은닉과 캡슐화에 대한 오해...

[객체지향의 올바른 이해 : 5. 정보 은닉(information hiding)](https://effectiveprogramming.tistory.com/entry/%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5-%EC%A0%95%EB%B3%B4-%EC%9D%80%EB%8B%89information-hiding%EC%97%90-%EB%8C%80%ED%95%9C-%EC%98%AC%EB%B0%94%EB%A5%B8-%EC%9D%B4%ED%95%B4)

[Information Hiding](http://egloos.zum.com/aeternum/v/1232020)

꼭 읽어보기

# 참고 자료

[Access Control - The Swift Programming Language (Swift 5.7)](https://docs.swift.org/swift-book/LanguageGuide/AccessControl.html)

[Swift_language_guide_kr/access-control.md at master · bbiguduk/Swift_language_guide_kr](https://github.com/bbiguduk/Swift_language_guide_kr/blob/master/language-guide-1/access-control.md)
