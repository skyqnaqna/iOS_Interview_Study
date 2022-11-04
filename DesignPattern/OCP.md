# Open/Colsed Principle 개방-폐쇄 원칙

> **기존의 코드를 변경하지 않고 기능을 수정하거나 추가할 수 있도록 설계해야 한다.**

## 적용방법
1. 기존 코드를 수정하는 대신 새 클래스를 만들어 붙이거나 상속 등을 통해 클래스 재사용
2. 변경(확장) 대상과 불변 대상을 명확히 구분한다. → 클래스 분리
3. 변경 대상과 불변 대상 사이에 인터페이스 정의
4. 구상 클래스(Concrete Class) 대신 인터페이스를 통해 코드 작성 → 전략 패턴 (Strategy Pattern)

## Ex1

<img src=https://user-images.githubusercontent.com/31722496/199668567-fcb6ec10-5ab4-4382-8533-08b29b8f6dd6.png width=50%>

<details>
<summary>Code</summary>
<div markdown="1">       

```swift
protocol Shooting {
    func shoot() -> String
}

// 레이저 빔. 쏠 수 있다.
final class LaserBeam: Shooting {
    func shoot() -> String {
        return "Ziiiiiip!"
    }
}

// 무기가 있고 모든 걸 한 번에 발사할 수 있다고 믿는다. 빵야! 빵야! 빵야!
final class WeaponsComposite {

    let weapons: [Shooting]

    init(weapons: [Shooting]) {
        self.weapons = weapons
    }

    func shoot() -> [String] {
        return weapons.map { $0.shoot() }
    }
}

let laser = LaserBeam()
var weapons = WeaponsComposite(weapons: [laser])

weapons.shoot()

// 로켓 런처. 로켓을 쏠 수 있다.
// ⚠️ 로켓 런처를 추가하기 위해 기존 클래스에서 아무것도 변경할 필요가 없다.
final class RocketLauncher: Shooting {
    func shoot() -> String {
        return "Whoosh!"
    }
}

let rocket = RocketLauncher()

weapons = WeaponsComposite(weapons: [laser, rocket])
weapons.shoot()
```

</div>
</details>

- 쏠 수 있는 무기를 추가할 때마다 기존 클래스에서 변경할 사항은 없다.
    - RocketLauncher라는 클래스를 추가할 때 기존 클래스에는 변경사항이 없다.

## 추가 질문

### 구상 클래스?

- 구상, 구체, 구현 (concrete, implementation) ↔ 추상 (abstract)
- 모든 메서드에 대한 구현이 되어 있는 클래스를 의미한다.
    - 인스턴스화할 수 있는, 추상 클래스가 아닌 클래스
- Abstract Class vs Concrete Class in Java → [https://www.geeksforgeeks.org/difference-between-abstract-class-and-concrete-class-in-java/?ref=rp](https://www.geeksforgeeks.org/difference-between-abstract-class-and-concrete-class-in-java/?ref=rp)

- 참고한 곳
	- [https://javacan.tistory.com/entry/구상-클래스란-용어의-어색함](https://javacan.tistory.com/entry/%EA%B5%AC%EC%83%81-%ED%81%B4%EB%9E%98%EC%8A%A4%EB%9E%80-%EC%9A%A9%EC%96%B4%EC%9D%98-%EC%96%B4%EC%83%89%ED%95%A8)
	- [https://www.geeksforgeeks.org/concrete-class-in-java/](https://www.geeksforgeeks.org/concrete-class-in-java/)

### 전략 패턴?

- 같은 종류의 작업을 하는 알고리즘을 정의 → 서로 다른 알고리즘들이 존재하고 실행 중 적합한 알고리즘을 선택해서 사용
- 클라이언트가 각 알고리즘을 바꿔서 적용시킬 수 있도록 한다.
- 모든 알고리즘이 동시에 사용되는 것이 아니면 굳이 함께 넣어야 할 필요가 없고, 클라이언트에 모든 알고리즘을 포함시키면 코드의 양이 늘어나고 복잡해진다. → 유지보수 어려움
- ex) 조리법이 다른 경우, 파일 압축 방식이 다른 경우 등

<img src=https://user-images.githubusercontent.com/31722496/199668959-eda6959e-03a9-4198-97f7-de29905244df.png width=50%>

- `Context` : 캡슐화된 알고리즘을 멤버 변수로 포함
- `Strategy` Interface : 컴파일 시점에서 사용하는 알고리즘을 나타냄. 실제 구현은 하위 클래스에 위임
- `Stratedy` Class : `Context` 에서 실행될 알고리즘 구현


### 캡슐화?

- 데이터와 데이터를 조작하는 함수를 결합하는 것 → 변수와 함수를 묶는 것
- 캡슐화를 통해 정보은닉과 데이터 추상화로 이어진다
- 정보은닉
    - 클래스의 데이터가 다른 클래스에서 숨겨지는 것
    - 클래스에서 접근 지정자를 사용하여 외부에서 볼 수 있는 데이터를 결정한다