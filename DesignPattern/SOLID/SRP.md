# Single Responsibility Principle 단일 책임 원칙

> **클래스는 하나의 기능(목적)을 가져야 한다.**  
>
>**클래스를 변경하는 이유는 하나여야 하고, 이를 지키지 않으면 코드가 길고 복잡해진다.**  
→ 유지보수에 비효율적

## 적용방법

1. 클래스 추출 (Extract Class)
    - 한 클래스 안에 구조를 변경하게 하는 이유가 둘 이상 존재하는 경우, 각 책임을 개별 클래스로 분할하여 클래스 당 하나의 책임만 맡도록 한다.
    - ex) 학생 클래스 : 성적정보, 재학 및 졸업정보, 등록금 납입정보
2. 슈퍼클래스 추출 (Extract Superclass)
    - 클래스를 나누고 보니 유사한 책임을 나눠 맡고 있는 경우, 공유되는 요소를 부모 클래스로 정의하여 부모 클래스에 위임한다.
    - ex) 대학생 성적정보, 대학원생 성적정보 클래스 : 성적정보 클래스를 상속하도록
3. 산탄총 수술 (Shotgun Surgery)
    - 하나에 여러가지 책임이 있는 것의 반대 상황으로 하나의 책임이 여러군데 분산되어 있는 상황
    - 흩어진 메소드들과 필드를 한 클래스로 합치거나 필요하다면 새 클래스를 만든다.
    - 산발적으로 여러 곳에 분포된 책임들을 한 곳에 모으면서 응집성을 높이는 작업이다. → High Cohesion

## Ex1

<img src=https://user-images.githubusercontent.com/31722496/199668047-bc769c4b-2020-4955-8dfe-de81202eeb47.png width=50%>

<details>
<summary>Code</summary>
<div markdown="1">       

```swift
protocol Openable {
	mutating func open()
}

protocol Closeable {
	mutating func close()
}

// 문. 캡슐화된 상태를 갖고 있으며 메서드를 사용해 변경할 수 있다.
struct PodBayDoor: Openable, Closeable {

	private enum State {
		case open
		case closed
	}

	private var state: State = .closed

	mutating func open() {
		state = .open
	}

	mutating func close() {
		state = .closed
	}
}

// 여는 일만 담당하며 안에 무엇이 들어있는 지, 어떻게 닫는 지 모른다.
final class DoorOpener {
	private var door: Openable

	init(door: Openable) {
		self.door = door
	}

	func execute() {
		door.open()
	}
}

// 닫는 일만 담당하며 안에 무엇이 들어있는 지, 어떻게 여는 지 모른다.
final class DoorCloser {
	private var door: Closeable

	init(door: Closeable) {
		self.door = door
	}

	func execute() {
		door.close()
	}
}

let door = PodBayDoor()

// ⚠️ `DoorOpeneer`만이 문을 여는 책임이 있다.
let doorOpener = DoorOpener(door: door)
doorOpener.execute()

// ⚠️ 문을 닫은 후 다른 작업을 해야 하는 경우,
// 알람을 켜는 것처럼 `DoorOpener` 클래스를 변경할 필요가 없다.
let doorCloser = DoorCloser(door: door)
doorCloser.execute()
```

</div>
</details>

- 문을 여는 일만 담당하는 `DoorOpener` 와 닫는 일만 담당하는 `DoorCloser` 클래스로 나눈다.
- 만약 문을 닫은 후 새로운 작업을 추가해야할 때, `DoorOpener` 클래스를 변경할 필요가 없다.