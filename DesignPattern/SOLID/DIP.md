# Dependency Inversion Principle 의존관계 역전 원칙

> **구체화에 의존하지 말고 추상화에 의존하라.**  
→ 의존 관계를 맺을 때 변하기 쉬운 것(구체적인 것) 보다는 변하기 어려운 것(추상적인 것)에 의존해야 한다.  
→ 구체화된 클래스에 의존하기 보다는 추상 클래스나 인터페이스에 의존해야 한다.
>
> **하위 모듈이 상위 모듈의 변경을 요구해서는 안 된다.**  
→ 상위 모듈이 하위 모듈의 구현에 의존해서는 안 된다.


- 할리우드 원칙, Inversion of Control, Dependency Injection 이라고도 함
- Caller가 Callee에 의존해야 할까, Callee가 Caller에 의존해야 할까?
    - 할리우드 배우(Callee)와 캐스팅 디렉터(Caller)가 있을 때
        - 디렉터가 교체되면 배우가 변경되어야 하는가?
        - 배우가 바뀌면 디렉터가 교체되어야 하는가?

## Ex1

<img src=https://user-images.githubusercontent.com/31722496/199675168-8a71018b-177b-49cc-aa5e-8fa142ad3967.png width=50%>

<details>
<summary>Code</summary>
<div markdown="1">       

```swift
protocol TimeTraveling {
    func travelInTime(time: TimeInterval) -> String
}

final class DeLorean: TimeTraveling {
	func travelInTime(time: TimeInterval) -> String {
		return "Used Flux Capacitor and travelled in time by: \(time)s"
	}
}

final class EmmettBrown {
	private let timeMachine: TimeTraveling

    // ⚠️ Emmet Brown은 `DeLorean`을 구체적인 클래스인 `DeLorean`이 아닌, `TimeTraveling` 장치로 받는다.
	init(timeMachine: TimeTraveling) {
		self.timeMachine = timeMachine
	}

	func travelInTime(time: TimeInterval) -> String {
		return timeMachine.travelInTime(time: time)
	}
}

let timeMachine = DeLorean()

let mastermind = EmmettBrown(timeMachine: timeMachine)
mastermind.travelInTime(time: -3600 * 8760)
```

</div>
</details>

- `DeLorean` : 영화 ‘백 투 더 퓨처’에 등장하는 자동차 기반 타임 머신 장치
- `EmmettBrown` : 영화 ‘백 투 더 퓨처’의 등장인물로 타임머신 발명자
- `Emmett Brown` 은 `timeMachine` 을 구체적인 클래스인 `DeLorean` 이 아닌, `TimeTraveling` 으로 받는다.

## Ex2

<img src=https://user-images.githubusercontent.com/31722496/199675397-af4a4b07-9f9b-4ce2-9147-732e34c549ea.png>

<img src=https://user-images.githubusercontent.com/31722496/199675467-0ff227c0-2d5b-4557-b7a6-b635855aba9a.png width=70%>

<details>
<summary>Code</summary>
<div markdown="1">       

```swift
class PDFReader {
  private var book: PDFBook?

  func construct(_ _book: PDFBook) {
    self.book = _book
  }

	func read() {
    self.book?.read()
  }
}

class PDFBook {
  func read() {
    print("Reading a pdf book")
  }
}
```

</div>
</details>

- 만약 `PDFBook` 의 `read()` 메소드가 변경된다면?
    - `PDFReader` 는 `PDFBook` 에 의존적이라고 할 수 있음

<img src=https://user-images.githubusercontent.com/31722496/199675641-aa9464d0-aea0-4e2b-af27-01e902d6cc7d.png width=70%>


<details>
<summary>Code</summary>
<div markdown="1">       

```swift
protocol EBook {
  func read()
}

class EBookReader: EBook {
  private var book: EBook?

  func construct(_ _book: EBook) {
    self.book = _book
  }

  func read() {
    self.book?.read()
  }
}

class PDFBook: EBook {
  func read() {
    print("Reading a pdf book")
  }
}

class EPUBBook: EBook {
  func read() {
    print("Reading a epub book")
  }
}
```

</div>
</details>

- `EBook` 이라는 추상화 계층을 삽입한다.
- 상위 모듈이 하위 모듈에 의존적이지 않도록 인터페이스를 참조하고, 하위 모듈이 바뀌어도 인터페이스를 만족시키면 되기 때문에 상위 모듈에는 영향을 주지 않는다.